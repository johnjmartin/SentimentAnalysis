#Sentiment Analysis
##Files included
  * fullOutput (output from mapreduce job)
  * reducedOutput
  * MyJob.jar
  * MyJob.java
  * README.md
  
##Execution
`./hadoop jar [path]/MyJob.jar /cisc432/negative.txt /cisc432/positive.txt /cisc432/data.txt [path]/final_output`
##MapReduce Programming & Development Process
My algorithm was to iterate over the bodies of every review for a product, matching each word to the provided list of positives and negatives. I kept a running counter starting at 0, adding 1 for each positive word matched, and subtracting 1 for each negative word. If there was more than one review for a product_id, I would carry over the value of the current count. This is essentially treats different reviews for a single product_id as one large review. I explain my thinking behind this later on. If after iterating through the whole review set the sum was > 0, = 0, or < 0 I decided the overall sentiment for the product was positive, neutral, or negative, respectively.

Translating this to MapReduce, for the mapper I read a single line of the file at a time. Assuming the line is in the expected format (there were several lines that were not and had to be skipped), I send the product_id to output as the key, and the review body as the value. This is a low computation process and happens very quickly even on the larger dataset. In the next phase, my reducer aggregates all of the reviews that have the same product_id into the Iterator<Text> values of the reducer. In the reducer I use some simple text processing to apply my above algorithm for sentiment analysis.

For job speed optimization, I tried several things. First I wrote my MapReduce so that the majority of computation happened within the reducer, where each partition has access to the list of bodies. This took approximately 3 minutes to run with the full dataset on Linux 4. Next, I tried to use a combiner at the end of each map, this made my code faster, but I was not able to get correct output from it. The combiner likely did not work because it did not have access to every single review body for a product and thus was not properly summing the count of each review body. Some details about my development and testing process with Hadoop.

I started testing using the reduced dataset from Assignment 2, in my mapper I knew I wanted the key to be product_id, and the value to be the review body. It took a bit of work to get my Mapper Headers implementing the correct types before my map class worked. When I thought I was done my mapper I ran:
“””./hadoop jar [path to jar] –D mapred.reduce.tasks=0 [input paths] [output path]’”””
to view the output of just my mapper job. Once the mapper output was what I was expecting, I started on my reducer.

My reducer took the product_id, and a Text Iterator for the review bodies. Using the iterator, I processed each text body to remove any symbols that were not characters or whitespace, then I split the review by whitespace, and calculated the total number of positives and negatives in the review body. I outputted “Positive Sentiment”, “Neutral Sentiment”, or “Negative Sentiment” depending on the total number of sentiment related words in each review. If there was more than one review I treated them as one large review, carrying the total sentiment count forward through each review iteration. I considered treating each review as +1 for positive, -1 for negative, and outputting that value. But it made more sense to me to take to count the total number of positive and negative words in each review because in that case if someone feels overwhelmingly for or against the product it is shown in the total count of the review sentiment.

Lastly, I definitely enjoyed this assignment the most. While I found MapReduce challenging to learn at first, things definitely speed up once you understand the basic structure of development. After seeing my final results, it’s clear how powerful MapReduce is for consuming, mutating, and analyzing large datasets. There were 1,431,031 different reviews in the dataset, the fact that it took less than 3 minutes to analyze and aggregate every review is very impressive and really shows the power of Hadoop.
