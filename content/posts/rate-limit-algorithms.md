+++
title = 'Rate Limit Algorithms'
date = 2024-06-13T23:29:07+08:00
draft = true
+++


## Fixed Window Algorithm

This algorithm uses a fixed window size that resets after a certain period. For example, if the window size is 100 requests and the window duration is one hour, then after 100 requests in less than an hour, all subsequent requests will be limited until the hour is up.

## Sliding Window Algorithm

This is an improvement over the fixed window algorithm. It uses a rolling time window to track the requests, which can prevent a burst of requests at the boundary of the window duration.


## Token Bucket Algorithm
This algorithm fills a 'bucket' with tokens at a fixed rate. Each request removes a token from the bucket. If the bucket is empty, the request is limited. This allows for bursty traffic as long as the average rate doesn't exceed the token fill rate

## Leaky Bucket Algorithm 
This algorithm is similar to the token bucket but it's designed to discard traffic once the limit is reached, like a leaky bucket. It's useful for smoothing out bursty traffic.

## GCRA (Generic Cell Rate Algorithm)

This algorithm is often used for network traffic shaping. It's similar to the leaky bucket algorithm but it also takes into account the time until the next possible request.