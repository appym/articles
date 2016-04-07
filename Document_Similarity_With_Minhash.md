In this blog post I am going to discuss about two probabilistic algorithms which are used for matching documents for similarities. Specifically, determining if two documents are similar to each other using Minhash. Comparing N documents for similarity using LSH. The implementation of Minhash and LSH algorithm are based on [1] and [2].

In this article similarity of documents do not mean contextual similarity rather lexical matching of documents.

#Similarity Techniques

##Jaccard Similarity
Jaccard similarity of two sets is the ratio of size of intersection of the two sets to the size of Union of the two sets.

For sets A and B

J (A, B) = |A ∩ B| / |A U B|

##Jaccard Bag Similarity 
Jaccard bag similarity for bags A and B is defined by counting an element n times in the intersection if n is the minimum of the number of times the element appears in A and B. In the union, count the sum of number of times an element appears in A and B.

Example – If bag A contains {a, a, a, b} and bag B contains {a, a, b, b, c}. Then min count of ‘a’ would be 2 and min count of ‘b’ would be 1 making a total of 3. The union of two bags is sum of sizes of each bag so 9. Hence Jaccard bag similarity would be 1/3

##Representation of documents

Before matching two documents for similarity, they need to be represented as sets. The most effective way to lexically match documents is to construct a set of N-Grams of the document. N-Grams or shingles are sets of N consecutive characters. So for example the 2-Gram for the word “match” would be {ma, at, tc, ch}. So it’s essentially a sliding window of 2 characters over the word “match”.

Shingling can also be constructed from words in a text instead of single characters. Shingles for a sentence would be a set formed from a sliding window of consecutive words. Example 2-shingles or 2-Grams for “Its quite sunny today” would be {“its quite”, “quite sunny”, “sunny today”}. This can be very helpful when matching documents that might have sentences in different order or words in sentences in different order. This can be customized to remove stop words from the documents.

Choosing the right shingle size is important. If the size were too small then the shingles would most likely repeat across documents leading to false positives. If the size were too large then the similarity would be very low leading to false negatives. The shingle size should be chosen such that probability of finding any given shingle in a document is low.

##Hashing Shingles

Another concern with shingles made from substrings of a document is the memory cost. In English there are 26 characters, but a document has other characters such as whitespace or ‘.’ or symbols which make the total possible shingles to be very high. A standard approximation for documents is to consider 20 characters (which mostly occur) and hence number of possible shingles would be 20^k for k-shingles.

To circumvent this cost, shingles for a large k, say 10 can be hashed to a bucket number in the 4-byte range thus compacting the data.

##Characteristic matrices and Signatures

If there were N documents being matched for similarity where N is a large number then it would not be feasible to match the many k-shingles for each document. Signature or smaller subsets are formed from each set of shingles (each set is derived from 1 document). Characteristic matrix represents the set or document identifier as columns and the elements of all the sets combined as rows. A cell is given the value 1 when an element is part of a set, 0 otherwise. Characteristic matrices are unlikely candidates for data storage as they are sparse matrices with more zeros than ones and are best used for visualizing data.

<table class="tg">
<tbody>
<tr>
<th class="tg-yw4l"></th>
<th class="tg-yw4l">D1</th>
<th class="tg-yw4l">D2</th>
<th class="tg-yw4l">....</th>
<th class="tg-yw4l">Dn</th>
</tr>
<tr>
<td class="tg-yw4l">h(s1)</td>
<td class="tg-yw4l">1</td>
<td class="tg-yw4l">0</td>
<td class="tg-yw4l">...</td>
<td class="tg-yw4l">1</td>
</tr>
<tr>
<td class="tg-yw4l">h(s2)</td>
<td class="tg-yw4l">0</td>
<td class="tg-yw4l">1</td>
<td class="tg-yw4l">...</td>
<td class="tg-yw4l">0</td>
</tr>
<tr>
<td class="tg-yw4l">...</td>
<td class="tg-yw4l">...</td>
<td class="tg-yw4l">...</td>
<td class="tg-yw4l">...</td>
<td class="tg-yw4l">...</td>
</tr>
<tr>
<td class="tg-yw4l">h(sn)</td>
<td class="tg-yw4l">1</td>
<td class="tg-yw4l">1</td>
<td class="tg-yw4l">...</td>
<td class="tg-yw4l">0</td>
</tr>
</tbody>
</table>

##Minhashing and Signature matrices

Minhashing is another way of representing the characteristic matrix mentioned above. Minhash of a set or document is a subset of say several hundred of the many hash functions applied to it. Minhash on a set is calculated by picking a permutation of rows for a column of the characteristic matrix  . The minhash value of any column would be the number of the first row in the permuted order where the column has a 1.

Signature matrices consist of the hash functions as rows and the sets as columns. Each column essentially is a vector of all the permuted hash values and is referred to as Minhash Signatures for that Set. Signature matrices are much smaller than characteristic matrices.

##Minhashing and Jaccard Similarity

To find the similarity between two documents Jaccard similarity of the minhash sets is a good estimator.

<table class="tg">
<tbody>
<tr>
<th class="tg-031e"></th>
<th class="tg-031e">D1</th>
<th class="tg-031e">D2</th>
</tr>
<tr>
<td class="tg-031e">h(s1)</td>
<td class="tg-031e">0</td>
<td class="tg-031e">0</td>
</tr>
<tr>
<td class="tg-031e">h(s2)</td>
<td class="tg-031e">1</td>
<td class="tg-031e">0</td>
</tr>
<tr>
<td class="tg-031e">...</td>
<td class="tg-031e">...</td>
<td class="tg-031e">...</td>
</tr>
<tr>
<td class="tg-031e">h(sn)</td>
<td class="tg-031e">1</td>
<td class="tg-031e">1</td>
</tr>
</tbody>
</table>
There are 3 types of combinations possible for the rows corresponding to each hash function as seen in the above table.

<ul>
	<li style="text-align: justify;">Both columns have 0's. Type X</li>
	<li style="text-align: justify;">One column has 0 and the other 1, Type Y</li>
	<li style="text-align: justify;">Both columns have 1's. Type Z</li>
</ul>
The Type X does not fall into either an union or an intersection operation on the two sets hence ignore them.

The union of the two sets would be combination of Y and Z type rows. Hence |A U B| = Y + Z

The intersection of two sets would be where both columns are 1 so type Z.

Now as mentioned before Jaccard Similarity is J (A, B) = |A ∩ B| / |A U B|

so J (A, B) = Z/ Y + Z

##Computing Minhash Signatures

Characteristics matrices are generally huge and picking a permutation of rows from millions of rows would be a time-consuming process hence a better way to do this is to use a random hash function which maps row numbers to bucket numbers. So its possible for multiple rows to be mapped to the same bucket number and cause collisions but if the number of buckets is large then the probability of collisions is much lower.

<table class="tg">
<tbody>
<tr>
<th class="tg-031e">Row</th>
<th class="tg-031e">S1</th>
<th class="tg-031e">S2</th>
<th class="tg-031e">S3</th>
<th class="tg-yw4l">S4</th>
<th class="tg-031e">h1(x+1 mod 5)</th>
<th class="tg-031e">h2 (3x + 1 mod 5)</th>
</tr>
<tr>
<td class="tg-031e">0</td>
<td class="tg-031e">1</td>
<td class="tg-031e">0</td>
<td class="tg-031e">0</td>
<td class="tg-yw4l">1</td>
<td class="tg-031e">1</td>
<td class="tg-031e">1</td>
</tr>
<tr>
<td class="tg-031e">1</td>
<td class="tg-031e">0</td>
<td class="tg-031e">0</td>
<td class="tg-031e">1</td>
<td class="tg-yw4l">0</td>
<td class="tg-031e">2</td>
<td class="tg-031e">4</td>
</tr>
<tr>
<td class="tg-031e">2</td>
<td class="tg-031e">0</td>
<td class="tg-031e">1</td>
<td class="tg-031e">0</td>
<td class="tg-yw4l">1</td>
<td class="tg-031e">3</td>
<td class="tg-031e">2</td>
</tr>
<tr>
<td class="tg-031e">3</td>
<td class="tg-031e">1</td>
<td class="tg-031e">0</td>
<td class="tg-031e">1</td>
<td class="tg-yw4l">1</td>
<td class="tg-031e">4</td>
<td class="tg-031e">0</td>
</tr>
<tr>
<td class="tg-yw4l">4</td>
<td class="tg-yw4l">0</td>
<td class="tg-yw4l">0</td>
<td class="tg-yw4l">1</td>
<td class="tg-yw4l">0</td>
<td class="tg-yw4l">0</td>
<td class="tg-yw4l">3</td>
</tr>
</tbody>
</table>
<p style="text-align: justify;">                                    <em>Table: Hash functions for Characteristic matrix</em></p>
<p style="text-align: justify;">To compute minhash signature matrix begin by choosing n (2 in above example) hash functions (h1 and h2) on the rows, the signature matrix initially consists of ∞ for all cells.</p>

<table class="tg">
<tbody>
<tr>
<th class="tg-baqh"></th>
<th class="tg-baqh">S1</th>
<th class="tg-baqh">S2</th>
<th class="tg-baqh">S3</th>
<th class="tg-yw4l">S4</th>
</tr>
<tr>
<td class="tg-6k2t">h1</td>
<td class="tg-j0tj">∞</td>
<td class="tg-j0tj">∞</td>
<td class="tg-j0tj">∞</td>
<td class="tg-j0tj">∞</td>
</tr>
<tr>
<td class="tg-yw4l">h2</td>
<td class="tg-baqh">∞</td>
<td class="tg-baqh">∞</td>
<td class="tg-baqh">∞</td>
<td class="tg-baqh">∞</td>
</tr>
</tbody>
</table>
<p style="text-align: justify;">First consider row 0, S1 and S4 have 1's and S2 and S3 have 0's so only S1 and S4 can be altered. The values of both h1(0) and h2(0) are 1 for row 0 and will replace ∞ (1 &lt; ∞). Thus the matrix looks like below.</p>

<table class="tg" style="height: 71px;" width="250">
<tbody>
<tr>
<th class="tg-baqh"></th>
<th class="tg-baqh">S1</th>
<th class="tg-baqh">S2</th>
<th class="tg-baqh">S3</th>
<th class="tg-yw4l">S4</th>
</tr>
<tr>
<td class="tg-6k2t">h1</td>
<td class="tg-j0tj">1</td>
<td class="tg-j0tj">∞</td>
<td class="tg-j0tj">∞</td>
<td class="tg-j0tj">1</td>
</tr>
<tr>
<td class="tg-yw4l">h2</td>
<td class="tg-baqh">1</td>
<td class="tg-baqh">∞</td>
<td class="tg-baqh">∞</td>
<td class="tg-baqh">1</td>
</tr>
</tbody>
</table>
<p style="text-align: justify;">Next for row 1, only S3 has 1 and can be altered. Thus we set the values of h1 and h2  for S3.</p>

<table class="tg" style="height: 71px;" width="250">
<tbody>
<tr>
<th class="tg-baqh"></th>
<th class="tg-baqh">S1</th>
<th class="tg-baqh">S2</th>
<th class="tg-baqh">S3</th>
<th class="tg-yw4l">S4</th>
</tr>
<tr>
<td class="tg-6k2t">h1</td>
<td class="tg-j0tj">1</td>
<td class="tg-j0tj">∞</td>
<td class="tg-j0tj">2</td>
<td class="tg-j0tj">1</td>
</tr>
<tr>
<td class="tg-yw4l">h2</td>
<td class="tg-baqh">1</td>
<td class="tg-baqh">∞</td>
<td class="tg-baqh">4</td>
<td class="tg-baqh">1</td>
</tr>
</tbody>
</table>
<p style="text-align: justify;">Similarly for row 2, has 1's for S2 and S4. S2 had ∞ for both h1 and h2 and will get replaced by 3 and 2 respectively. S4 had 1's for both h1 and h2 which is lower than 3 and 2 hence there will be no change.</p>

<table class="tg" style="height: 71px;" width="250">
<tbody>
<tr>
<th class="tg-baqh"></th>
<th class="tg-baqh">S1</th>
<th class="tg-baqh">S2</th>
<th class="tg-baqh">S3</th>
<th class="tg-yw4l">S4</th>
</tr>
<tr>
<td class="tg-6k2t">h1</td>
<td class="tg-j0tj">1</td>
<td class="tg-j0tj">3</td>
<td class="tg-j0tj">2</td>
<td class="tg-j0tj">1</td>
</tr>
<tr>
<td class="tg-yw4l">h2</td>
<td class="tg-baqh">1</td>
<td class="tg-baqh">2</td>
<td class="tg-baqh">4</td>
<td class="tg-baqh">1</td>
</tr>
</tbody>
</table>
Following through the minhash process of iteratively replacing a hash value if there exists an hash value lower than it we get the signature matrix as below.
<table class="tg" style="height: 71px;" width="250">
<tbody>
<tr>
<th class="tg-baqh"></th>
<th class="tg-baqh">S1</th>
<th class="tg-baqh">S2</th>
<th class="tg-baqh">S3</th>
<th class="tg-yw4l">S4</th>
</tr>
<tr>
<td class="tg-6k2t">h1</td>
<td class="tg-j0tj">1</td>
<td class="tg-j0tj">3</td>
<td class="tg-j0tj">0</td>
<td class="tg-j0tj">1</td>
</tr>
<tr>
<td class="tg-yw4l">h2</td>
<td class="tg-baqh">0</td>
<td class="tg-baqh">2</td>
<td class="tg-baqh">0</td>
<td class="tg-baqh">0</td>
</tr>
</tbody>
</table>

Jaccard Similarity can be applied to the above signature matrix to determine similarity between sets (documents). So for example the Jaccard similarity between S1 and S2 would be 0 (hashes don't match) whereas for s1 and s3 it would be 0.5 (one hash matches).

#Locality Sensitive Hashing

Minhash solves the problem of comparing large documents by compressing them into signature matrices which are a much smaller dataset compared to the original characteristic matrix. But when there are a large number of large documents involved then comparing them to each other would still be a time consuming task because of the number of pairs of documents that would need to be compared for similarity.

Locality sensitive hashing or near neighbor search solves this problem by focussing on pairs which are likely to be similar rather than on all pairs.

The idea is to hash each item several times such that similar items would more likely hash to the same bucket than dissimilar items. Any pair of documents which hashed to the same bucket is treated as a candidate pair and these are in turn evaluated for similarity either using Jaccard rule or cosine distances or euclidean approach.

##LSH with Minhash 

If we have already computed the minhash signatures for the items then an effective to LSH would be to divide the signature matrix into b bands of r rows each. For each band there is a hash function that takes the vector of r values and hashes them to various buckets.

Pick a threshold value t such that t = (1/b) ^ (1/r). Amongst the candidate pairs find out which pairs have Jaccard similarity higher than t to get the list of documents which are most similar to each other.

You can find the code worked for matching two documents here:

https://github.com/appym/lsh

##References

1. Rajaraman, Anand and Jeff Ullman. 2010 (Draft). Mining of Massive Data Sets(Chapter 3).
2. http://okomestudio.net/biboroku/?p=2065
