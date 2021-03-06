---
layout: page
mathjax: true
permalink: /2016/projects/p19/midterm/
---

## 电影推荐系统-中期报告

### 一．	数据收集与表示

1.电影编码如下：

	12,Finding Nemo (2003)"
	13,Forrest Gump (1994)"
	14,American Beauty (1999)"
	22,Pirates of the Caribbean: The Curse of the Black Pearl (2003)"
	24,Kill Bill: Vol. 1 (2003)"
	38,Eternal Sunshine of the Spotless Mind (2004)"
	63,Twelve Monkeys (a.k.a. 12 Monkeys) (1995)"
	77,Memento (2000)"
	85,Raiders of the Lost Ark (Indiana Jones and the Raiders of the Lost Ark) (1981)"
	98,Gladiator (2000)"
	105,Back to the Future (1985)"
	107,Snatch (2000)"
	114,Pretty Woman (1990)"
	120,The Lord of the Rings: The Fellowship of the Ring (2001)"
	121,The Lord of the Rings: The Two Towers (2002)"
	122,The Lord of the Rings: The Return of the King (2003)"
	134,O Brother Where Art Thou? (2000)"
	141,Donnie Darko (2001)"
	146,Crouching Tiger Hidden Dragon (Wo hu cang long) (2000)"

2.用户对电影评分如下


3.电影的属性



### 二．	建立模型处理数据

1. 计算个体-标签矢量（模型）
  在模型构建器中实现，为数据集中的每部电影计算单位标准化TF-IDF载体。该模型包含每一项从ID到TF-IDF向量的映射，这一映射标准化为单位向量

```java

TFIDFModelget() {

	Map<String, Long> tagIds = buildTagIdMap();

	MutableSparseVector docFreq = MutableSparseVector.create(tagIds.values());
	docFreq.fill(0);

	Map<Long,MutableSparseVector> itemVectors = Maps.newHashMap();

	MutableSparseVector work = MutableSparseVector.create(tagIds.values());

	// Iterate over the items to compute each item's vector.
	LongSet items = dao.getItemIds();
	long counter = 0;
	for (long item: items) {
	    counter++;
	    // Reset the work vector for this item's tags.
	    work.clear();

	    List<String> tags =  dao.getItemTags(item);
	    for(String tag: tags){

	        long tagId = tagIds.get(tag);

	        try{
	            work.set(tagId, work.get(tagId)+1);
	        }catch(IllegalArgumentException e){
	            // init value to 1
	            work.set(tagId, 1);
	        }
	    }

	    MutableSparseVector temp = MutableSparseVector.create(tagIds.values());

	    for(String tag: tags){
	        long tagId = tagIds.get(tag);
	        try{
	            // we already saw this tag in this item.
	            temp.get(tagId);
	            continue;
	        }catch(IllegalArgumentException e){
	            // first time we see this tag in this item.
	            temp.set(tagId, 1);
	            docFreq.set(tagId, docFreq.get(tagId)+1);
	        }

	    }

	    // Save a shrunk copy of the vector (only storing tags that apply to this item) in
	    // our map, we'll add IDF and normalize later.
	    itemVectors.put(item, work.shrinkDomain());

			for (VectorEntry e: docFreq.fast()) {

	      long tagId = e.getKey();
	      if(tagId ==1){
	          System.out.println("");
	      }
	      double idf = e.getValue();
	      double log_idf = Math.log10(counter/idf);
	      docFreq.set(tagId, log_idf);

	    }

	    Map<Long,SparseVector> modelData = Maps.newHashMap();
	    for (Map.Entry<Long,MutableSparseVector> entry: itemVectors.entrySet()) {
	        MutableSparseVector tv = entry.getValue(); // tv is the TF of each tag for this item
	        // TODO Convert this vector to a TF-IDF vector
	        for(VectorEntry v: tv.fast()){

	          double tf = v.getValue();
	          //System.out.print(" tf: ");
	          //System.out.print(tf);


	          double idf = docFreq.get(v.getKey());
	          //System.out.print(" idf: ");
	          //System.out.print(idf);

	          tv.set(v.getKey(), tf*idf);
	          //System.out.print(" tf*idf ");
	          //System.out.println(tv.get(v.getKey()));
	    		}

	    		double len = tv.norm();


	        for(VectorEntry v: tv.fast()){
	            tv.set(v.getKey(), v.getValue()/len);
	        }

	        // Store a frozen (immutable) version of the vector in the model data.
	        modelData.put(entry.getKey(), tv.freeze());
				}

	// we technically don't need the IDF vector anymore, so long as we have no new tags
	return new TFIDFModel(tagIds, modelData);

}
```

2.为每个查询用户建立用户档案

makeUserVector方法由输入的用户ID产生代表该用户的个人档案的向量。在最初的实现方案中，该配置文件是用户评为所有项的个体-标签矢量中大于等于3.5星的数量之和。后来我们使用了加权用户配置文件对原有方案进行改善。加权信息由所有项的项矢量加权和计算出来，其权值基于用户的评分。

```java
makeUserVector(user) {
    // Get the user's ratings
    List<Rating> userRatings = dao.getEventsForUser(user, Rating.class);
    if (userRatings == null) {
        // the user doesn't exist
        return SparseVector.empty();
    }
}
```

### 三.向用户推荐

正在进行，尚未完成！
