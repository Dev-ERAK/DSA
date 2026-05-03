# DSA Interview Question Bank

This repository contains structured Data Structures & Algorithms interview questions.
Each question must have its own Markdown explanation file.

---

# Mandatory Coding Rule — Use Java AND Python

For every DSA problem you solve:

You MUST implement the solution in:

1. Java
2. Python

This ensures:

- Language flexibility in interviews
- Strong understanding of logic
- Ability to switch languages quickly
- Better debugging skills

---

# Repository Rule

Every problem file MUST contain:

- Java solution
- Python solution
- Time & Space Complexity
- Edge Cases
- Explanation

# Folder Structure (Recommended)

```
dsa-prep/
│
├── README.md
├── arrays/
│   ├── rotate-array.md
│   ├── kadane-algorithm.md
│
├── strings/
│   ├── palindrome-check.md
│   ├── longest-palindrome.md
│
├── linked-list/
├── stack-queue/
├── trees/
├── graphs/
├── dynamic-programming/
├── hashing/
├── recursion-backtracking/
├── greedy/
├── searching/
└── mathematics/
```

---

# 1. Intro to DSA

## Algorithms & Data Structures

1. Design a data structure that supports insert, delete, and getRandom in O(1).
2. Explain the difference between Array, Linked List, Stack, and Queue with use cases.

## Time & Space Complexity

1. What is the time complexity of nested loops where the inner loop runs from i to n?
2. Explain time vs space trade-off with an example.

## Big-O Notation

1. Rank the following in increasing order:
   O(n), O(log n), O(n log n), O(n²)

2. Why is binary search O(log n)?

---

# 2. Arrays

## 1D / 2D Arrays

1. Rotate an array by k steps.
2. Search a number in a sorted 2D matrix.

## Prefix Sum

1. Find the sum of elements between two indices efficiently.
2. Count subarrays with sum equal to k.

## Sliding Window

1. Find the maximum sum subarray of size k.
2. Find the longest substring without repeating characters.

## Kadane’s Algorithm

1. Find the maximum subarray sum.
2. Find the maximum circular subarray sum.

## Sorting

1. Implement merge sort.
2. Find the kth largest element.

---

# 3. Strings

## Palindrome

1. Check if a string is a palindrome ignoring special characters.
2. Find the longest palindromic substring.

## Anagram

1. Check if two strings are anagrams.
2. Group words into anagram groups.

## String Matching

1. Implement substring search.
2. Find the first non-repeating character.

---

# 4. Linked Lists

## Singly

1. Reverse a linked list.
2. Find the middle node.

## Cycle Detection

1. Detect cycle using slow and fast pointers.
2. Find the start of the cycle.

## Merge Point

1. Find intersection of two linked lists.
2. Find length difference method.

---

# 5. Stack & Queue

## Stack

1. Evaluate postfix expression.
2. Implement stack using arrays.

## Queue

1. Implement queue using linked list.
2. Implement queue using stacks.

## Min Stack

1. Design a stack that returns minimum in O(1).
2. Explain auxiliary stack logic.

---

# 6. Hashing

## HashMap

1. Solve the Two Sum problem.
2. Find frequency of elements.

## HashSet

1. Remove duplicates from array.
2. Check if array contains duplicates.

---

# 7. Recursion & Backtracking

## Fibonacci

1. Implement Fibonacci recursively.
2. Optimize using memoization.

## Subsets

1. Generate all subsets.
2. Generate subsets without duplicates.

## Permutations

1. Generate all permutations.
2. Generate unique permutations.

---

# 8. Trees

## Binary Tree

1. Find height of binary tree.
2. Check if tree is balanced.

## BST

1. Insert into BST.
2. Validate BST.

## LCA

1. Find lowest common ancestor in BST.
2. Find LCA in binary tree.

---

# 9. Dynamic Programming

## LCS

1. Find longest common subsequence.
2. Print LCS string.

## LIS

1. Find longest increasing subsequence.
2. Optimize LIS to O(n log n).

## 0/1 Knapsack

1. Solve knapsack using DP.
2. Solve using memoization.

---

# 10. Graphs

## BFS

1. Traverse graph using BFS.
2. Find shortest path in unweighted graph.

## DFS

1. Traverse graph using DFS.
2. Detect cycle in graph.

## Dijkstra

1. Find shortest path using Dijkstra.
2. Why does Dijkstra fail with negative edges?

## DSU

1. Implement union-find.
2. Detect cycle using DSU.

---

# 11. Greedy

## Activity Selection

1. Select maximum non-overlapping activities.
2. Explain why greedy works.

## Coin Change

1. Minimum coins problem.
2. When greedy fails?

---

# 12. Searching

## Linear Search

1. Find element in array.
2. Find first occurrence.

## Binary Search

1. Implement binary search.
2. Find square root using binary search.

## Binary Search on Answer

1. Find minimum capacity to ship packages.
2. Find minimum eating speed.

---

# 13. Mathematics

## GCD / LCM

1. Find GCD using Euclidean algorithm.
2. Find LCM of two numbers.

## Prime Numbers

1. Check if a number is prime.
2. Count primes less than n.

## Bit Manipulation

1. Check if number is power of two.
2. Count number of set bits.

---

### Notes

You must create ONE Markdown file per question
This is the discipline rule.
If you skip this, progress will be slow.

Here is the mandatory template you should use.

# Problem: Rotate Array by K Steps

## Category

Arrays

## Difficulty

Medium

## Problem Statement

Given an array of integers and an integer k, rotate the array to the right by k steps.

Example:

Input:
nums = [1,2,3,4,5,6,7]
k = 3

Output:
[5,6,7,1,2,3,4]

---

## Clarifying Questions

- Can k be larger than array size?
- Can array contain negative numbers?
- Can array be empty?

---

## Approach

Method 1: Brute Force
Rotate one step at a time.

Method 2: Reverse Algorithm (Optimal)

Steps:

1. Reverse entire array
2. Reverse first k elements
3. Reverse remaining elements

---

## Time Complexity

O(n)

---

## Space Complexity

O(1)

---

## Code (Java)

```java
public void rotate(int[] nums, int k) {
    k = k % nums.length;

    reverse(nums, 0, nums.length - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, nums.length - 1);
}

private void reverse(int[] arr, int start, int end) {
    while (start < end) {
        int temp = arr[start];
        arr[start] = arr[end];
        arr[end] = temp;
        start++;
        end--;
    }
}

```

---

## Code (Python)

```Python

class Solution:

    def example(self, nums):

        # logic here

        return 0
```

---

## Edge Cases

- k = 0
- k > array length
- empty array
- single element

---

## Follow-up Questions

- Can this be done using extra space?
- Can this be done using linked list?
