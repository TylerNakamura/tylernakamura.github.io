---
layout: post
title: "Testing Concurrent Map Operations in Go"
date: 2018-07-17
comments: false
published: True
categories: go concurrency maps
---

## The Problem
Imagine you had a big map (or dict for your Pythonistas)
This map stored many custom structs with many numerical values.
Now imagine you had a queue of transactions waiting to mutate that map.
Some transactions reduced values in the map, while others increased them.
How do you process the transactions?

The naive solution is to process the queue one by one.

But we want to be fast! :)
So of course, we have many goroutines to process our queue.
The problem is, what happens when two goroutines want to mutate a value within the same key?
Duhh, race condition.
Our next naive solution is to place a mutex lock on the map.
By placing a mutex lock for the map, we can ensure that no two goroutines access it at the same time.

"But we want it to be faster!"
Okay, okay.
My next trick is to place a lock on _each_ key of the map.
Because the value of this map is a custom struct, we can create a unique lock for each of the keys.
That way, many keys can be mutated at the same time.

This article aims to look at these three approaches, measure them, and show my findings.
Spoiler alert, solution 3 is the fastest.

## A Sample Implementation
// create our custom struct
// each struct has its own lock
// create a map of our custom structs
// create a list of transactions
// get a correct answer
// do the same work in parallel
// create a queue
// populate queue
// run many threads to go through the queue
// each thread needs to sleep for a tiny bit (jitter) to pretend like it's working
// compare the correct answer with the parallel answer

```go
package main

import (
	"fmt"
	"sync"
	"time"
	"math/rand"
)

// let's pretend we are tracking many warehouses that have a count of these random items
type warehouse struct {
	// note that we have a read write mutex for EACH warehouse
	rwlock sync.RWMutex

	// unique id
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

	targetWarehouse int

	balesOfHay int
	porkChops int
	waterBottles int
	gadgets int
	gizmos int
}

func main() {
	// the map that keeps track of all of our warehouses
	warehouses1 := make(map[int]*warehouse)
	warehouses2 := make(map[int]*warehouse)

	numberOfWarehouses := 100
	numberOfTransactions := 5000

	maxMillisecondJitter := 4

	// use a buffered channel as a queue
	// we have two because, we are going to go through them twice, once with multi threads, and another time serialized
	queue1 := make(chan warehouseTransaction, numberOfTransactions)
	queue2 := make(chan warehouseTransaction, numberOfTransactions)

	// generate new seed: https://gobyexample.com/random-numbers
	s := rand.NewSource(time.Now().UnixNano())
	r := rand.New(s)

	// populate warehouses with many random warehouse structs
	fmt.Println("Populating Warehouse Map")
	for i := 0; i < numberOfWarehouses; i++ {
		temp1 := createRandomizedWarehouse(i, r)
		temp2 := temp1
		warehouses1[i] = &temp1
		warehouses2[i] = &temp2
	}

	fmt.Println("Populating Queues with Random Warehouse Transactions")
	// populate the queues with random warehouse transactions
	for i := 0; i < numberOfTransactions; i++ {
		temp := createRandomizedWarehouseTransaction(numberOfWarehouses, r)
		// push random transaction onto both queues
		queue1 <- temp
		queue2 <- temp
	}

	//  ================================= SERIAL PROCCESSING ===================================
	fmt.Println("Processing Queue1 with Serial Processing")
	// start a timer
	serialStart := time.Now()
	for i := 0; i < numberOfTransactions; i++ {
		transactWithoutLock(warehouses1, <-queue1, r, maxMillisecondJitter)
	}
	serialElapsed := time.Since(serialStart)

	//  ================================= CONCURRENT PROCESSING =================================
	fmt.Println("Processing Queue2 with Concurrent Processing")
	// start a timer
	concurrentStart := time.Now()
	var wg sync.WaitGroup
	wg.Add(numberOfTransactions)
	for i := 0; i < numberOfTransactions; i++ {
		go func() {
			defer wg.Done()
			transactWithLock(warehouses2, <-queue2, r, maxMillisecondJitter)
		}()
	}
	wg.Wait()
	concurrentElapsed := time.Since(concurrentStart)


	//  ================================= CHECK RESULTS =======================================
	// we want to make sure that our concurrent results were able to achieve the same end result as the serial results
	fmt.Println()
	if !warehousesAreEqual(warehouses1, warehouses2) {
		fmt.Println("FAILURE: The warehouse maps did not have the same end state, something went wrong in the processing")
	} else {
		fmt.Println("=======================================================")
		fmt.Printf("Total Number of Warehouses:      %d\n", numberOfWarehouses)
		fmt.Printf("Total Number of Transactions:    %d\n", numberOfTransactions)
		fmt.Printf("Seconds to Process in Serial:    %v\n", serialElapsed.Seconds())
		fmt.Printf("Seconds to Process Concurrently: %v\n", concurrentElapsed.Seconds())
		fmt.Println("=======================================================")
	}
}

func warehousesAreEqual(whMap1 map[int]*warehouse, whMap2 map[int]*warehouse) bool{
	for k, v := range(whMap1) {
		if v.balesOfHay != whMap2[k].balesOfHay {
			fmt.Printf("WarehouseMap1 Warehouse #%d has %d balesOfHay, WarehouseMap2 Warehouse #%d has %d balesOfHay\n", k, v.balesOfHay, k, whMap2[k].balesOfHay)
			return false
		}
		if v.porkChops != whMap2[k].porkChops {
			fmt.Printf("WarehouseMap1 Warehouse #%d has %d porkChops, WarehouseMap2 Warehouse #%d has %d porkChops\n", k, v.porkChops, k, whMap2[k].porkChops)
			return false
		}
		if v.waterBottles != whMap2[k].waterBottles {
			fmt.Printf("WarehouseMap1 Warehouse #%d has %d waterBottles, WarehouseMap2 Warehouse #%d has %d waterBottles\n", k, v.waterBottles, k, whMap2[k].waterBottles)
			return false
		}
		if v.gadgets != whMap2[k].gadgets {
			fmt.Printf("WarehouseMap1 Warehouse #%d has %d gadgets, WarehouseMap2 Warehouse #%d has %d gadgets\n", k, v.gadgets, k, whMap2[k].gadgets)
			return false
		}
		if v.gizmos != whMap2[k].gizmos {
			fmt.Printf("WarehouseMap1 Warehouse #%d has %d gizmos, WarehouseMap2 Warehouse #%d has %d gizmos\n", k, v.gizmos, k, whMap2[k].gizmos)
			return false
		}
	}
	return true
}

func transactWithoutLock(whMap map[int]*warehouse, transaction warehouseTransaction, r *rand.Rand, maxMillisecondJitter int){
	// pretend like we are doing some work, like network/disk io
	jitter := r.Intn(maxMillisecondJitter)
	time.Sleep(time.Duration(jitter) * time.Millisecond)

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

func transactWithLock(whMap map[int]*warehouse, transaction warehouseTransaction, r *rand.Rand, maxMillisecondJitter int){
	// pretend like we are doing some work, like network/disk io
	jitter := r.Intn(maxMillisecondJitter)
	time.Sleep(time.Duration(jitter) * time.Millisecond)

	// NOTE: this transaction is identical to transactWithoutLock, except in this case we block to acquire the lock
	// you will notice that if you comment out these locking lines, you will end up with race conditions and your resulting maps are not the same!
	// this is the behaviour we expect :D
	whMap[transaction.targetWarehouse].rwlock.Lock()
	defer whMap[transaction.targetWarehouse].rwlock.Unlock()

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
func createRandomizedWarehouse(id int, r *rand.Rand)warehouse{
	temp := warehouse{}
	temp.id = id
	temp.balesOfHay = r.Intn(10000)
	temp.porkChops = r.Intn(10000)
	temp.waterBottles = r.Intn(10000)
	temp.gadgets = r.Intn(10000)
	temp.gizmos = r.Intn(10000)
	return temp
}

func createRandomizedWarehouseTransaction(numberOfWarehouses int, r *rand.Rand)warehouseTransaction{
	temp := warehouseTransaction{}
	// I want to randomize whether or not this is an increase or a decrease, this is my lazy way of coming up with a boolean, I'm sure there is a better way to do it
	temp.increase = (r.Intn(10) + 1) % 2 == 0

	// randomly select one of the warehouses to apply this transaction to
	temp.targetWarehouse = r.Intn(numberOfWarehouses)

	temp.balesOfHay = r.Intn(100)
	temp.porkChops = r.Intn(100)
	temp.waterBottles = r.Intn(100)
	temp.gadgets = r.Intn(100)
	temp.gizmos = r.Intn(100)
	return temp
}

func prettyPrintWarehouse(input warehouse){
	fmt.Println(fmt.Sprintf("Warehouse #%d - Bales of Hay: %d, Pork Chops: %d, Water Bottles: %d, Gadgets: %d, Gizmos: %d", input.id, input.balesOfHay, input.porkChops, input.waterBottles, input.gadgets, input.gizmos))
}

```

Intel(R) Core(TM) i7-4870HQ CPU @ 2.50GHz
