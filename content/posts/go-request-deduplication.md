+++
title = 'Go Deduplication of function call'
date = 2024-06-30T10:47:31+08:00
draft = true
+++

## Implement deduplication of request with channels


```golang
func (c Cache) Cache(key string, generator CacheGenerator) (string, error) {
	response := make(chan CacheResponse)
	req := CacheReuest{
		key:       key,
		generator: generator,
		response:  response,
	}

	c.requests <- req
    resp := <-response

	return resp.value, resp.err
}


// Handler that will be execuited 
func (c Cache) requestHandler() {
	requestStorage := map[string][]CacheReuest{}

	for {
		select {
		case req := <-c.requests:
			reqQ, ok := requestStorage[req.key]

            if ok {
                requestStorage[req.key] = append(reqQ, req)
				continue
            }

            requestStorage[req.key] = []CacheReuest{req}

			go c.processRequest(req)
        case resp := <-c.responses:
			reqQ, ok := requestStorage[resp.key]

			if !ok {
				continue
			}

			for _, req := range reqQ {
				req.response <- resp
			}

			delete(requestStorage, resp.key)
        case <-c.ctx.Done():
           //Here some clean up logic
        }
    }
}   
```

# Implementing with using  singleflight groups

```golang
import (
    ...
    "golang.org/x/sync/singleflight"
)

type Cache struct{
    ...
    sfGroup *singleflight.Group
}

func (c Cache) Cache(key string, generator CacheGenerator) (string, error) {
	resultChan := resc.sfGroup.DoChan(key, func() (interface{}, error) {
	    return c.process(key, generator)
    })

    res <- resultChan

    return res.Val, res.Err
}

```
