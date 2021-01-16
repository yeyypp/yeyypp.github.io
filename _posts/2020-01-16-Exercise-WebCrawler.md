---
title: "Exercise Web Crawler"
date: 2021-01-16
---
# [Exercise: Web Crawler](https://tour.golang.org/concurrency/10)
Using WaitGroup to make sure every goroutine finish their job.
```
package main

import (
	"fmt"
	"sync"
)

type Memo struct {
	wg sync.WaitGroup
	mu sync.Mutex
	memo map[string]bool
	}
func (m Memo) check(url string) bool {
	m.mu.Lock()
	defer m.mu.Unlock()
	if _, ok := m.memo[url]; ok {
		return true
		}
		m.memo[url] = true
		return false
		}
		
var m  = Memo{memo: make(map[string]bool)}

type Fetcher interface {
	// Fetch returns the body of URL and
	// a slice of URLs found on that page.
	Fetch(url string) (body string, urls []string, err error)
}

// Crawl uses fetcher to recursively crawl
// pages starting with url, to a maximum of depth.
func Crawl(url string, depth int, fetcher Fetcher) {
	// TODO: Fetch URLs in parallel.
	// TODO: Don't fetch the same URL twice.
	// This implementation doesn't do either:
	defer m.wg.Done()
	if depth <= 0 {
		return
	}
	if m.check(url) {
		return
		}
	body, urls, err := fetcher.Fetch(url)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("found: %s %q\n", url, body)
	for _, u := range urls {
		m.wg.Add(1)
		go Crawl(u, depth-1, fetcher)
	}
	return
}

func main() {
	m.wg.Add(1)
	Crawl("https://golang.org/", 4, fetcher)
	m.wg.Wait()
}

// fakeFetcher is Fetcher that returns canned results.
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher is a populated fakeFetcher.
var fetcher = fakeFetcher{
	"https://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"https://golang.org/pkg/",
			"https://golang.org/cmd/",
		},
	},
	"https://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"https://golang.org/",
			"https://golang.org/cmd/",
			"https://golang.org/pkg/fmt/",
			"https://golang.org/pkg/os/",
		},
	},
	"https://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
	"https://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
}
```
