# Coding Practices & Interview Problems

## 📌 Additional Resources
- Must Read [Here](https://github.com/nithinyell/Quick-Samples)

---

## Common Coding Interview Problems

### 1. Second Highest Integer
Find the second highest integer in an array.

```swift
func findSecondHighest(in array: [Int]) -> Int? {
    guard array.count >= 2 else { return nil }
    
    var highest = Int.min
    var secondHighest = Int.min
    
    for num in array {
        if num > highest {
            secondHighest = highest
            highest = num
        } else if num > secondHighest && num != highest {
            secondHighest = num
        }
    }
    
    return secondHighest == Int.min ? nil : secondHighest
}

// Using sorted (less efficient O(n log n))
func findSecondHighestSimple(in array: [Int]) -> Int? {
    let unique = Array(Set(array)).sorted(by: >)
    return unique.count >= 2 ? unique[1] : nil
}

// Example
let numbers = [10, 5, 20, 8, 20, 15]
print(findSecondHighest(in: numbers)) // 15
```

---

### 2. Limit the Number of Occurrences in an Array
Remove elements from array that appear more than a specified number of times.

```swift
func limitOccurrences(_ array: [Int], maxOccurrences: Int) -> [Int] {
    var countDict: [Int: Int] = [:]
    var result: [Int] = []
    
    for num in array {
        let count = countDict[num, default: 0]
        if count < maxOccurrences {
            result.append(num)
            countDict[num] = count + 1
        }
    }
    
    return result
}

// Example
let nums = [1, 2, 2, 3, 3, 3, 4, 5, 5]
print(limitOccurrences(nums, maxOccurrences: 2)) // [1, 2, 2, 3, 3, 4, 5, 5]
```

---

### 3. Number of Times a Word is Repeated in String
Count occurrences of a word in a string.

```swift
func countWordOccurrences(in text: String, word: String) -> Int {
    let words = text.lowercased().components(separatedBy: .whitespacesAndNewlines)
    return words.filter { $0 == word.lowercased() }.count
}

// Alternative using reduce
func countWordOccurrencesReduce(in text: String, word: String) -> Int {
    let words = text.lowercased().components(separatedBy: .whitespacesAndNewlines)
    return words.reduce(0) { $1 == word.lowercased() ? $0 + 1 : $0 }
}

// Example
let sentence = "Swift is great and Swift is fun. I love Swift!"
print(countWordOccurrences(in: sentence, word: "Swift")) // 3
```

---

### 4. Find Subarray that Makes Sum Zero
Find if there's a contiguous subarray with sum equal to zero.

```swift
func hasZeroSumSubarray(_ array: [Int]) -> Bool {
    var sum = 0
    var sumSet: Set<Int> = [0] // Include 0 for subarrays starting from index 0
    
    for num in array {
        sum += num
        if sumSet.contains(sum) {
            return true
        }
        sumSet.insert(sum)
    }
    
    return false
}

// Find the actual subarray
func findZeroSumSubarray(_ array: [Int]) -> [Int]? {
    var sum = 0
    var sumMap: [Int: Int] = [0: -1]
    
    for (index, num) in array.enumerated() {
        sum += num
        
        if let startIndex = sumMap[sum] {
            return Array(array[(startIndex + 1)...index])
        }
        
        sumMap[sum] = index
    }
    
    return nil
}

// Example
let arr = [4, 2, -3, 1, 6]
print(hasZeroSumSubarray(arr)) // true
print(findZeroSumSubarray(arr) ?? []) // [2, -3, 1]
```

---

## Additional Common Problems

### 5. Reverse a String
```swift
func reverseString(_ str: String) -> String {
    return String(str.reversed())
}

// Without using reversed()
func reverseStringManual(_ str: String) -> String {
    var chars = Array(str)
    var left = 0
    var right = chars.count - 1
    
    while left < right {
        chars.swapAt(left, right)
        left += 1
        right -= 1
    }
    
    return String(chars)
}
```

### 6. Check if String is Palindrome
```swift
func isPalindrome(_ str: String) -> Bool {
    let cleaned = str.lowercased().filter { $0.isLetter || $0.isNumber }
    return cleaned == String(cleaned.reversed())
}

// Example
print(isPalindrome("A man a plan a canal Panama")) // true
```

### 7. Find Duplicate Elements in Array
```swift
func findDuplicates(_ array: [Int]) -> [Int] {
    var seen = Set<Int>()
    var duplicates = Set<Int>()
    
    for num in array {
        if seen.contains(num) {
            duplicates.insert(num)
        } else {
            seen.insert(num)
        }
    }
    
    return Array(duplicates)
}
```

### 8. Two Sum Problem
```swift
func twoSum(_ nums: [Int], _ target: Int) -> [Int]? {
    var dict: [Int: Int] = [:]
    
    for (index, num) in nums.enumerated() {
        let complement = target - num
        if let complementIndex = dict[complement] {
            return [complementIndex, index]
        }
        dict[num] = index
    }
    
    return nil
}

// Example
print(twoSum([2, 7, 11, 15], 9) ?? []) // [0, 1]
```

### 9. FizzBuzz
```swift
func fizzBuzz(n: Int) -> [String] {
    return (1...n).map { num in
        switch (num % 3 == 0, num % 5 == 0) {
        case (true, true):
            return "FizzBuzz"
        case (true, false):
            return "Fizz"
        case (false, true):
            return "Buzz"
        default:
            return "\(num)"
        }
    }
}
```

### 10. Remove Duplicates from Sorted Array
```swift
func removeDuplicates(_ nums: inout [Int]) -> Int {
    guard !nums.isEmpty else { return 0 }
    
    var writeIndex = 1
    
    for readIndex in 1..<nums.count {
        if nums[readIndex] != nums[readIndex - 1] {
            nums[writeIndex] = nums[readIndex]
            writeIndex += 1
        }
    }
    
    return writeIndex
}
```

---

## 💡 Tips for Coding Interviews
1. **Clarify the problem** - Ask questions about edge cases
2. **Think out loud** - Explain your approach
3. **Start with brute force** - Then optimize
4. **Consider time and space complexity** - Big O notation
5. **Test your code** - Walk through examples
6. **Handle edge cases** - Empty arrays, nil values, negative numbers

