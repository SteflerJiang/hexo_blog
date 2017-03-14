---
title: Lucene索引内存结构
date: 2017-03-10 11:47:23
categories: Solr
tags: Solr, Lucene
---

Lucene索引在内存中的结构主要分为三个部分: field, term, docId，具体如下。

# Field在内存中的组织结构
field->fileposition，域名对应在文件中的偏移量，`TreeMap<String,Long> fields = new TreeMap<>();`.

具体代码在org.apache.lucene.codecs.simpletext.SimpleTextFieldsReader中的readFields()方法中。
```java
  private TreeMap<String,Long> readFields(IndexInput in) throws IOException {
    ChecksumIndexInput input = new BufferedChecksumIndexInput(in);
    BytesRefBuilder scratch = new BytesRefBuilder();
    TreeMap<String,Long> fields = new TreeMap<>();

    while (true) {
      SimpleTextUtil.readLine(input, scratch);
      if (scratch.get().equals(END)) {
        SimpleTextUtil.checkFooter(input);
        return fields;
      } else if (StringHelper.startsWith(scratch.get(), FIELD)) {
        String fieldName = new String(scratch.bytes(), FIELD.length, scratch.length() - FIELD.length, StandardCharsets.UTF_8);
        fields.put(fieldName, input.getFilePointer());
      }
    }
  }
```
这里存放的value是偏移量。{content:234,keyword:456,title:767}

# Term在内存中的组织结构
定义了一个有限状态机`private FST<PairOutputs.Pair<Long,PairOutputs.Pair<Long,Long>>> fst;`

fst的input即term的值，output有三个内容：
- term在倒排表中的偏移量，即在倒排表文件中的位置
- docFreq，即这个term在多少个文档中出现
- totalTermFreq，即这个term在文档中出现的总次数

具体源码在org.apache.lucene.codecs.simpletext.SimpleTextFieldsReader中的loadTerms()方法中。

```java
    private void loadTerms() throws IOException {
      PositiveIntOutputs posIntOutputs = PositiveIntOutputs.getSingleton();
      final Builder<PairOutputs.Pair<Long,PairOutputs.Pair<Long,Long>>> b;
      final PairOutputs<Long,Long> outputsInner = new PairOutputs<>(posIntOutputs, posIntOutputs);
      final PairOutputs<Long,PairOutputs.Pair<Long,Long>> outputs = new PairOutputs<>(posIntOutputs,
          outputsInner);
      b = new Builder<>(FST.INPUT_TYPE.BYTE1, outputs);
      // 打开倒排文件表
      IndexInput in = SimpleTextFieldsReader.this.in.clone();
      // 定位文件中的位置，这个值就是上面介绍的field保存的偏移量
      in.seek(termsStart);
      final BytesRefBuilder lastTerm = new BytesRefBuilder();
      long lastDocsStart = -1;
      int docFreq = 0;
      long totalTermFreq = 0;
      FixedBitSet visitedDocs = new FixedBitSet(maxDoc);
      final IntsRefBuilder scratchIntsRef = new IntsRefBuilder();
      while(true) {
        SimpleTextUtil.readLine(in, scratch);
        if (scratch.get().equals(END) || StringHelper.startsWith(scratch.get(), FIELD)) {
            // 读取结束的时候，也把最后一个保存起来，并且保证第一次读到field的时候，lastDocsStart为-1，这时候不保存
          if (lastDocsStart != -1) {
            b.add(Util.toIntsRef(lastTerm.get(), scratchIntsRef),
                outputs.newPair(lastDocsStart,
                    outputsInner.newPair((long) docFreq, totalTermFreq)));
            sumTotalTermFreq += totalTermFreq;
          }
          break;
        } else if (StringHelper.startsWith(scratch.get(), DOC)) {
          docFreq++;
          sumDocFreq++;
          scratchUTF16.copyUTF8Bytes(scratch.bytes(), DOC.length, scratch.length()-DOC.length);
          int docID = ArrayUtil.parseInt(scratchUTF16.chars(), 0, scratchUTF16.length());
          visitedDocs.set(docID);
        } else if (StringHelper.startsWith(scratch.get(), FREQ)) {
          scratchUTF16.copyUTF8Bytes(scratch.bytes(), FREQ.length, scratch.length()-FREQ.length);
          totalTermFreq += ArrayUtil.parseInt(scratchUTF16.chars(), 0, scratchUTF16.length());
        } else if (StringHelper.startsWith(scratch.get(), TERM)) {
            //读取到term的时候，把数据保存到fst中
          if (lastDocsStart != -1) {
            //往FST数据结构中添加term数据,lastTerm是term的字节数组,lastDocsStart是这个term在倒排表文件中的偏移量,docFreq是出现这个term的文档数,totalTermFreq是这个term在这些文档中总共出现的次数
            b.add(Util.toIntsRef(lastTerm.get(), scratchIntsRef), outputs.newPair(lastDocsStart,
                outputsInner.newPair((long) docFreq, totalTermFreq)));
          }
          lastDocsStart = in.getFilePointer();
          final int len = scratch.length() - TERM.length;
          lastTerm.grow(len);
          System.arraycopy(scratch.bytes(), TERM.length, lastTerm.bytes(), 0, len);
          lastTerm.setLength(len);
          docFreq = 0;
          sumTotalTermFreq += totalTermFreq;
          totalTermFreq = 0;
          termCount++;
        }
      }
      docCount = visitedDocs.cardinality();
      fst = b.finish();
      /*
      PrintStream ps = new PrintStream("out.dot");
      fst.toDot(ps);
      ps.close();
      System.out.println("SAVED out.dot");
      */
      //System.out.println("FST " + fst.sizeInBytes());
    }
```

# docId在内存中的组织结构
docid -> offset，`private long offsets[]; /* docid -> offset in .fld file */`，文档id在索引文件（存储文档数据的文件）中的偏移量。

具体的数据结构是：[12,23,37,77......]，数组的下标就是文档id，值就是文档在文件中的偏移量。

比如我们要取回文档0的值，那我们根据这个数组找到这个文档在文件中的偏移量是12，然后用随机读定位到文件的这个位置，接着就是read方法取回数据了.源代码在org.apache.lucene.codecs.simpletext.SimpleTextStoredFieldsReader的visitDocument()方法。

```java
  public void visitDocument(int n, StoredFieldVisitor visitor) throws IOException {
    in.seek(offsets[n]);
    
    while (true) {
      readLine();
      if (StringHelper.startsWith(scratch.get(), FIELD) == false) {
        break;
      }
      int fieldNumber = parseIntAt(FIELD.length);
      FieldInfo fieldInfo = fieldInfos.fieldInfo(fieldNumber);
      readLine();
      assert StringHelper.startsWith(scratch.get(), NAME);
      readLine();
      assert StringHelper.startsWith(scratch.get(), TYPE);
      
      final BytesRef type;
      if (equalsAt(TYPE_STRING, scratch.get(), TYPE.length)) {
        type = TYPE_STRING;
      } else if (equalsAt(TYPE_BINARY, scratch.get(), TYPE.length)) {
        type = TYPE_BINARY;
      } else if (equalsAt(TYPE_INT, scratch.get(), TYPE.length)) {
        type = TYPE_INT;
      } else if (equalsAt(TYPE_LONG, scratch.get(), TYPE.length)) {
        type = TYPE_LONG;
      } else if (equalsAt(TYPE_FLOAT, scratch.get(), TYPE.length)) {
        type = TYPE_FLOAT;
      } else if (equalsAt(TYPE_DOUBLE, scratch.get(), TYPE.length)) {
        type = TYPE_DOUBLE;
      } else {
        throw new RuntimeException("unknown field type");
      }
      
      switch (visitor.needsField(fieldInfo)) {
        case YES:  
          readField(type, fieldInfo, visitor);
          break;
        case NO:   
          readLine();
          assert StringHelper.startsWith(scratch.get(), VALUE);
          break;
        case STOP: return;
      }
    }
  }
```

# Lucene搜索时，索引在文件和内存中的状态
现在用一个搜索过程来看下如何加载和使用这些数据,我们使用的搜索条件是：title=hello

流程图如下
[lucene索引流程图](/images/Lucene索引流程.png "Lucene索引流程")
