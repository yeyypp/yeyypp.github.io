# [Quick Sort](https://en.wikipedia.org/wiki/Quicksort#Implementation_issues)
- Lomuto partition scheme
This scheme degrades to O(n2) when the array is already in order
```
func QuickSort(nums []int) {
  sort(nums []int, 0, len(nums) - 1)
}

func sort(nums []int, low, high int) {
  if low < high {
    j := partition(nums, low, high)
    sort(nums, low, j - 1)
    sort(nums, j + 1, high)
    }
}

func partition(nums []int, low, high int) int {
  key, i, j := nums[low], low, high
  for {
    for nums[i] <= key {
      i++
      if i == high {
        break
        }
     }
    for nums[j] >= key {
      j--
      if j == low {
        break
        }
     }
     if i >= j {
        break
        }
     swap(nums, i, j)
    }
    swap(nums, low, j)
    return j
}

func swap(nums []int, i, j int) {
  nums[i], nums[j] = nums[j], nums[i]
  }
```
- Hoare partition scheme
```

```
