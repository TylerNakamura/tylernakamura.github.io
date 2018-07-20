---
layout: post
title: "Concurrent Map Operations in Go"
date: 2018-07-17
comments: false
published: True
categories: go concurrency maps mutex sync
---

## The Problem
You have map.
This map stored many custom structs with many numerical values.
We also have queue of transactions waiting to mutate values in the map.
Some transactions reduced values in the map, while others increased them.
How might you process transactions in the fastest manner?

<img src="/images/2018/07/general.png"/>

The naive approach is to process the queue one by one.

<img src="/images/2018/07/approach1.png"/>

But we want to be fast! :)
So of course, we could spawn multiple Goroutines to process our queue.
The problem is, what happens when two Goroutines want to mutate a value within the same key at the same time?
We will most likely have a race condition of course!
Our next naive approach is to place a mutex lock on the map.
By placing a mutex lock for the map, we can ensure that no two Goroutines access a given key/value at the same time.

<img src="/images/2018/07/approach2.png"/>

"But we want it to be faster!"
Okay, okay.
My next trick is to place a lock on _each_ key of the map.
We can use a custom struct to represent the values of our map and place a unique mutex on each value.
This way, many different keys can be mutated at the same time.

<img src="/images/2018/07/approach3.png"/>

This article aims to look at these three approaches, measure them, and share my findings.
Spoiler alert, approach 3 is the fastest.

## Implementation
I wanted to see what each of these looked like in code.
It also served as a good opportunity for me to practice some concurrency in Go.
My hypothesis is that approach 3 would be the fastest, but I wanted to create a demo and see it for myself.

```go
package main

import (
	"fmt"
	"sync"
	"time"
	"math/rand"
)

// a fictitious object representing a warehouse
type warehouse struct {
	// NOTE: read/write mutex for EACH warehouse
	rwlock sync.RWMutex

	// unique identifier for this warehouse
	id int

	// characteristics
	balesOfHay int
	porkChops int
	waterBottles int
	gadgets int
	gizmos int
}

// represents a single transaction against a warehouse
type warehouseTransaction struct {
	// if true, the warehouse should ADD the below characteristics to it's total stores
	// otherwise, the warehouse inventory is being deducted
	increase bool

	// that ID of the warehouse that the transaction should be acted against
	targetWarehouse int

	balesOfHay int
	porkChops int
	waterBottles int
	gadgets int
	gizmos int
}

func main() {
	// VARIABLES
	numberOfWarehouses := 10000
	numberOfTransactions := 1000000
	millisecondsToProcess := 0

	warehouseMap1 := make(map[int]*warehouse)
	warehouseMap2 := make(map[int]*warehouse)
	warehouseMap3 := make(map[int]*warehouse)
	queue1 := make(chan warehouseTransaction, numberOfTransactions)
	queue2 := make(chan warehouseTransaction, numberOfTransactions)
	queue3 := make(chan warehouseTransaction, numberOfTransactions)

	// generate new seed: https://gobyexample.com/random-numbers
	s := rand.NewSource(time.Now().UnixNano())
	r := rand.New(s)

	var wg sync.WaitGroup

	//  ================================= POPULATE RANDOM DATA ========================
	fmt.Println("Populating Warehouse Maps with Random Data")
	for i := 0; i < numberOfWarehouses; i++ {
		temp1 := createRandomizedWarehouse(i, r)
		temp2 := temp1
		temp3 := temp1
		warehouseMap1[i] = &temp1
		warehouseMap2[i] = &temp2
		warehouseMap3[i] = &temp3
	}
	fmt.Println("Populating Queues with Random Transactions")
	for i := 0; i < numberOfTransactions; i++ {
		temp := createRandomizedWarehouseTransaction(numberOfWarehouses, r)
		queue1 <- temp
		queue2 <- temp
		queue3 <- temp
	}

	//  ================================= SERIAL PROCCESSING ============================
	fmt.Println("Processing Queue1 with Serial Processing")
	// start a timer
	queue1Start := time.Now()
	for i := 0; i < numberOfTransactions; i++ {
		processTransaction(warehouseMap1, <-queue1, r, millisecondsToProcess, false)
	}
	queue1Duration := time.Since(queue1Start)

	//  ================================= CONCURRENT PROCESSING (map lock) ===============
	fmt.Println("Processing Queue2 with Map Locking")
	// start a timer
	queue2Start := time.Now()
	// a mutex lock to lock the entire map
	var mutex = &sync.Mutex{}
	wg.Add(numberOfTransactions)
	for i := 0; i < numberOfTransactions; i++ {
		go func() {
			// lock the entire map
			mutex.Lock()
			defer func() {
				wg.Done()
				mutex.Unlock()
			}()
			processTransaction(warehouseMap2, <-queue2, r, millisecondsToProcess, false)
		}()
	}
	// block until all the goroutines are done
	wg.Wait()
	queue2Duration := time.Since(queue2Start)

	//  ================================= CONCURRENT PROCESSING (key lock) ==============
	fmt.Println("Processing Queue3 with Key Locking")
	// start a timer
	queue3Start := time.Now()
	wg.Add(numberOfTransactions)
	for i := 0; i < numberOfTransactions; i++ {
		go func() {
			defer wg.Done()
			processTransaction(warehouseMap3, <-queue3, r, millisecondsToProcess, true)
		}()
	}
	// block until all the goroutines are done
	wg.Wait()
	queue3Duration := time.Since(queue3Start)

	//  ================================= RESULTS =======================================
	// we want to make sure that our concurrent results were able to achieve the same end result as the serial results
	fmt.Println()
	if !warehousesAreEqual(warehouseMap1, warehouseMap2) {
		fmt.Println("FAILURE: The Map1 and Map2 did not have the same end state, something went wrong in the processing")
	}
	if !warehousesAreEqual(warehouseMap1, warehouseMap3) {
		fmt.Println("FAILURE: The Map1 and Map3 did not have the same end state, something went wrong in the processing")
	}

	fmt.Println("========================VARIABLES======================")
	fmt.Printf("Total Number of Warehouses:            %d\n", numberOfWarehouses)
	fmt.Printf("Total Number of Transactions:          %d\n", numberOfTransactions)
	fmt.Printf("Milliseconds to Process a Transaction: %d\n", millisecondsToProcess)
	fmt.Println("=======================================================")
	fmt.Println("==========================RESULTS======================")
	fmt.Printf("Milliseconds to Process (Serial):      %v\n", queue1Duration.Seconds() * 1000)
	fmt.Printf("Milliseconds to Process (Map Lock):    %v\n", queue2Duration.Seconds() * 1000)
	fmt.Printf("Milliseconds to Process (Key Lock):    %v\n", queue3Duration.Seconds() * 1000)
	fmt.Println("=======================================================")
}

// evaluates two warehouse objects to determine if they share the same exact
func warehousesAreEqual(whMap1 map[int]*warehouse, whMap2 map[int]*warehouse) bool{
	for k, v := range(whMap1) {
		if v.balesOfHay != whMap2[k].balesOfHay {
			fmt.Printf("Warehouse #%d has %d balesOfHay and Warehouse #%d has %d balesOfHay. The warehouse maps are not equal!\n", k, v.balesOfHay, k, whMap2[k].balesOfHay)
			return false
		}
		if v.porkChops != whMap2[k].porkChops {
			fmt.Printf("Warehouse #%d has %d porkChops and Warehouse #%d has %d porkChops. The warehouse maps are not equal!\n", k, v.porkChops, k, whMap2[k].porkChops)
			return false
		}
		if v.waterBottles != whMap2[k].waterBottles {
			fmt.Printf("Warehouse #%d has %d waterBottles and Warehouse #%d has %d waterBottles. The warehouse maps are not equal!\n", k, v.waterBottles, k, whMap2[k].waterBottles)
			return false
		}
		if v.gadgets != whMap2[k].gadgets {
			fmt.Printf("Warehouse #%d has %d gadgets and Warehouse #%d has %d gadgets. The warehouse maps are not equal!\n", k, v.gadgets, k, whMap2[k].gadgets)
			return false
		}
		if v.gizmos != whMap2[k].gizmos {
			fmt.Printf("Warehouse #%d has %d gizmos and Warehouse #%d has %d gizmos. The warehouse maps are not equal!\n", k, v.gizmos, k, whMap2[k].gizmos)
			return false
		}
	}
	if len(whMap1) != len(whMap2) {
		fmt.Printf("Warehouses are not equal in length")
		return false
	}
	return true
}

func processTransaction(whMap map[int]*warehouse, transaction warehouseTransaction, r *rand.Rand, millisecondsToProcess int, useKeyLock bool){
	if useKeyLock {
		// you will notice that if you comment out these locking lines, you will end up with race conditions and your resulting maps will not be the same
		whMap[transaction.targetWarehouse].rwlock.Lock()
		defer whMap[transaction.targetWarehouse].rwlock.Unlock()
	}

	time.Sleep(time.Duration(millisecondsToProcess) * time.Millisecond)

	// if it's an increase type transaction, add resources to the target warehouse
	if transaction.increase {
		whMap[transaction.targetWarehouse].balesOfHay += transaction.balesOfHay
		whMap[transaction.targetWarehouse].porkChops += transaction.porkChops
		whMap[transaction.targetWarehouse].waterBottles += transaction.waterBottles
		whMap[transaction.targetWarehouse].gadgets += transaction.gadgets
		whMap[transaction.targetWarehouse].gizmos += transaction.gizmos
	} else { // otherwise, deduct from the target warehouse
		whMap[transaction.targetWarehouse].balesOfHay -= transaction.balesOfHay
		whMap[transaction.targetWarehouse].porkChops -= transaction.porkChops
		whMap[transaction.targetWarehouse].waterBottles -= transaction.waterBottles
		whMap[transaction.targetWarehouse].gadgets -= transaction.gadgets
		whMap[transaction.targetWarehouse].gizmos -= transaction.gizmos
	}
}

// creates and returns a single warehouse struct with randomized data
func createRandomizedWarehouse(id int, r *rand.Rand) warehouse{
	temp := warehouse{}
	temp.id = id
	temp.balesOfHay = r.Intn(10000)
	temp.porkChops = r.Intn(10000)
	temp.waterBottles = r.Intn(10000)
	temp.gadgets = r.Intn(10000)
	temp.gizmos = r.Intn(10000)
	return temp
}

// creates and returns a single warehouse transaction struct with randomized data
func createRandomizedWarehouseTransaction(numberOfWarehouses int, r *rand.Rand) warehouseTransaction{
	temp := warehouseTransaction{}

	temp.increase = (r.Intn(2) + 1) % 2 == 0

	// randomly select one of the warehouses to apply this transaction to
	temp.targetWarehouse = r.Intn(numberOfWarehouses)

	temp.balesOfHay = r.Intn(100)
	temp.porkChops = r.Intn(100)
	temp.waterBottles = r.Intn(100)
	temp.gadgets = r.Intn(100)
	temp.gizmos = r.Intn(100)
	return temp
}

// for debugging purposes, will print out a warehouse struct in a nice format
func prettyPrintWarehouse(input warehouse){
	fmt.Printf("Warehouse #%d - Bales of Hay: %d, Pork Chops: %d, Water Bottles: %d, Gadgets: %d, Gizmos: %d\n", input.id, input.balesOfHay, input.porkChops, input.waterBottles, input.gadgets, input.gizmos)
}
```

## Results

1 Warehouse (no concurrency available):
```bash
========================VARIABLES======================
Total Number of Warehouses:            1
Total Number of Transactions:          1000
Milliseconds to Process a Transaction: 2
=======================================================
==========================RESULTS======================
Milliseconds to Process (Serial):      2539.431903
Milliseconds to Process (Map Lock):    2306.711409
Milliseconds to Process (Key Lock):    2314.826846
=======================================================
```

2 Warehouses (minimal concurrency available):
```bash
========================VARIABLES======================
Total Number of Warehouses:            2
Total Number of Transactions:          1000
Milliseconds to Process a Transaction: 2
=======================================================
==========================RESULTS======================
Milliseconds to Process (Serial):      2392.9029210000003
Milliseconds to Process (Map Lock):    2366.987177
Milliseconds to Process (Key Lock):    1203.4260100000001
=======================================================
```

Many Warehouses (lots of concurrency available):
```bash
========================VARIABLES======================
Total Number of Warehouses:            1000
Total Number of Transactions:          1000
Milliseconds to Process a Transaction: 2
=======================================================
==========================RESULTS======================
Milliseconds to Process (Serial):      2476.1787520000003
Milliseconds to Process (Map Lock):    2461.1052760000002
Milliseconds to Process (Key Lock):    16.631619999999998
=======================================================
```

## Conclusion
It was fun to fiddle with each of the variables and watch the results dance.
I especially loved learning about a pattern in which a channel could be used as a queue.
You can find it in the code above, but it really just looks like this:
```go
// declare a queue of things to process using a buffered channel
myQueue := make(chan thingToProcess, size)
// add things onto the queue
myQueue <- thingToProcess
// process things off the queue by reading from the channel
processItem(<-myQueue)
```

TLDR: *By placing mutex locks on each of the keys, instead of the entire map, we were able to increase our concurrency depending on the number of keys.*

## CPU
Intel(R) Core(TM) i7-4870HQ CPU @ 2.50GHz
