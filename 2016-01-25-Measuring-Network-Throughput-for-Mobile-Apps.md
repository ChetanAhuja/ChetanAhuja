---
title: Measuring Network Throughput for Mobile Apps
layout: post
author: chetan
permalink: /measuring-network-throughput-for-mobile-apps/
source-id: 1jXe9Bi7YvGVhhXy8YZwd6745ui9i-RLO_RcEaqzA4mA
published: true
---
**Measuring Network Throughput for Mobile Apps**

If your application has downloadable content and you care about how fast that content is getting delivered to your users, one of the first things you should do is to accurately measure the throughput seen by your app. Now, this looks as simple as measuring the time taken for each of your network requests and dividing your data size by the time. While that holds true, there are some cases where that may not be good enough. 

 

In the following post, we will discuss how to think about throughput as it relates to mobile apps, and also how the assumptions that many people are make can lead to misinterpreting data. We will use:

 

    D for data transferred (bytes), 

    T for time taken (seconds) and 

    S for speed or throughput (bytes per second). 

 

 Let us consider a few cases for which we want to measure throughput and various pitfalls that are involved. 

 

**Case 1:** 1 transfer of 1000 bytes that finished in 2 seconds. 

    S = D/T  = 1000/2 = 500 bytes per second. 

 ![image alt text]({{ site.url }}/public/CzNN9CeCWnK99vZBcXkGCQ_img_0.jpg)

There is absolutely nothing wrong with this measurement. If your app does exactly 1 transfer then this would be a correct way to measure, however we have found that most apps in the real world do not stop at a solitary transfer.

Case 1 Throughput = 500 bytes / second

 

 

**Case 2:** 2 serial transfers of 1000 and 500 bytes that finished in 2 and 1 seconds respectively. (By serial, we mean second transfer starts after the first one ended as shown in the image below) 

 ![image alt text]({{ site.url }}/public/CzNN9CeCWnK99vZBcXkGCQ_img_1.jpg)

Here, we could measure the overall speed in two ways. Let N be the number of transfers, 

** **

**Method A: **S = (S1+S2+...+Sn)/N  

**Method B: **S = (D1+D2+...+Dn)/(T1+T2+...+Tn)

 

Now, for the above Case 2, we get the following speeds..

Method A: S = (500/1 + 1000/2)/2 = 500 bytes per second. 

Method B: S = (500+1000)/(1+2) = 500 bytes per second. 

 

Here, Method A says your speed is 500 bytes per second which means you get 1500 bytes per 3 seconds while Method B also says the speed is 500 bytes per second which is the same 1500 bytes for 3 seconds.   So these two methods must be the same, right?

 

No.  Method A is actually calculating the *average of your speeds*, not the *average speed*.  This is a very common mistake.  Your overall speed is not the average of your speeds. Method A is obviously wrong and you should always use Method B. 

Case 2 Throughput  = 500 bytes / second

 

In both Case 1 and Case 2, we have determined that the throughput of our imaginary system is 500 bytes / second.  Let us consider a different case that is very common among many mobile apps. 

**Case 3 **

Let's take a very simple (and simplistic) situation as an example. An app needs to download 8 files of 10KB bytes each from a single server using the HTTP protocol.  This would be a common occurrence in an app displaying a bunch of thumbnail images which is typical of many retail apps. A typical HTTP client side stack would create 4 simultaneous connections to a server. We can (simplistically) assume that all 4 of these connections start and end exactly at the same time, as shown in the diagram. 

![image alt text]({{ site.url }}/public/CzNN9CeCWnK99vZBcXkGCQ_img_2.jpg)

**Method A: **S 	= ((10/3)+(10/3)+(10/3)+(10/3)+(10/3)+(10/3)+(10/3)+(10/3))/8

=(26.7)/8

=3.33 KB/S

  

**Method B: **S 	= (D1+D2+...+Dn)/(T1+T2+...+Tn)

		= (10+10+10+10+10+10+10+10)/((3+3+3+3+3+3+3+3)

		= 80/24

		= 3.33 KB/S

As soon as the first 4 transfers are complete, the second batch of 4 transfers will be triggered by the system. For the sake of simplicity, let's assume this entire second batch also finishes exactly at the same time (this would be normally a bad assumption for this case with 4 competing TCP connections but that's a topic for a different post). 

 Let's contrast this with the case where all 8 transfers were started together in one big batch..

![image alt text]({{ site.url }}/public/CzNN9CeCWnK99vZBcXkGCQ_img_3.jpg)

**Method A: **S 	= ((10/6)+(10/6)+(10/6)+(10/6)+(10/6)+(10/6)+(10/6)+(10/6))/8

=(13.3)/8

=1.67 KB/S

  

**Method B: **S 	= (D1+D2+...+Dn)/(T1+T2+...+Tn)

		= (10+10+10+10+10+10+10+10)/((6+6+6+6+6+6+6+6)

		= 1.67 KB/S

10 * 8 / 6*8 (Using method B)

80 / 6 (Using our method) - Which is same as above

 Note how exactly the same situation (8 transfers donw in 6 seconds) resulted in two extremely different calculations.

**Case 4:** 2 parallel transfers of 1000 and 500 bytes that finished in 3 and 2 seconds respectively. (By parallel, we mean both the transfers started at once as shown in the image below) 

![image alt text]({{ site.url }}/public/CzNN9CeCWnK99vZBcXkGCQ_img_4.jpg)

 

We will go ahead with just the Method B from Case 2.

**Method B: **S = (D1+D2+...+Dn)/(T1+T2+...+Tn)

S = (1000+500)/(3+2) = 300 bytes per second. 

 

Is this right?  (*spoiler*: No) While Method B is fine for calculating throughput for a series of downloads, it fails us when we are looking at requests in parallel.  

Think of it another way - the underlying reason that you care about the throughput for a mobile app is likely because of how the throughput affects the user experience.  If that is the case, then the total time needed to download the files is actually the metric that is important.

Let's revisit Case 2: Two requests start at time 0 and finish at 3 and 2 seconds.  

 - this means that we had received 1000+500 bytes of data *by the end* of 3 seconds. So, the throughput should have been 1500/3 = 500 bytes per second. 

If you look back to Case 1 and Case 2, you'll see that a 1000 bytes took 2 s and a 500 bytes took 1 second.  The reason why Case 3 had different time for each of these is because when you have parallel transfers, you are by definition sharing the available bandwidth.  For the first 2 seconds of the transfer in Case 3, both files are each using 250 bytes/second.  When the 500 byte file is completed at t=2, 500 bytes of the 1000 byte file has also been received and 500 bytes remain. At t=2, there is suddenly a lot more available bandwidth for the rest of the second file and is therefore able to send the second half of the file in 1 additional second.

![image alt text]({{ site.url }}/public/CzNN9CeCWnK99vZBcXkGCQ_img_5.jpg)

Let's call this Method C which is applicable when you have parallel or overlapping transfers. 

 

**Method C:** 

D = (total size of your transfers),  

T = (end timestamp of last transfer - start timestamp of first transfer) 

S = D/ T

 

**Note:** Your overall speed cannot be determined with just transfer size and elapsed time. You need to consider when and where the transfers start and end too. 

The challenge is that it's very hard for app developers to know in advance which of their transfers will happen in parallel so it’s quite impractical, if not impossible to carry out direct measurements for total bandwidth obtained for any arbitrary interval during an app’s lifetime. Most available tools (New Relic etc.) are content to simply assume that each network transfer (typically an HTTP request) is completely independent. This leads to highly misleading data made available to the developers and performance engineers. This just means more confusion and wasted hours when trying to compare results from A/B tests of various optimizations.

The first step towards accurate measurements is to split all the transfers from a session into multiple non-overlapping groups (G) as shown in the below image. Once you have a bunch of non overlapping groups, consider that group as a single transfer and calculate D and T for that group transfer using Method C. Then, use Method B on the group transfers to get a reasonably correct throughput estimate. 

 

![image alt text]({{ site.url }}/public/CzNN9CeCWnK99vZBcXkGCQ_img_6.jpg)

 

If we use this formula, we don't have to worry about the request patterns, as you will get the same result in all the above three cases using this.

 Our method is great… etc…

The hard question here is, if it's impractical to try and make direct bandwidth measurements in presence of arbitrary number of parallel requests during any given interval, how do we arrive at a satisfactory metric for network performance of the app? 

Faced with the same problem, the PacketZoom engineering team came up with a solution. This solution is easy to apply if all you have available is individual measurements of network transfers, along with start times of each of the transfers.  I'll discuss this method in the next post.

