```js
function mergeSort(nums) {
  function merge(left, right, arr) {
    let i = 0,
      j = 0,
      k = 0;
    while (i < left.length && j < right.length) {
      if (left[i] < right[j]) {
        arr[k++] = left[i++];
      } else {
        arr[k++] = right[j++];
      }
    }
    while (i < left.length) arr[k++] = left[i++];
    while (j < right.length) arr[k++] = right[j++];
  }
  function sort(arr) {
    if (arr.length < 2) return;
    const mid = Math.floor(arr.length / 2);
    const left = arr.slice(0, mid);
    const right = arr.slice(mid);
    sort(left);
    sort(right);
    merge(left, right, arr);
  }
  sort(nums);
  return nums;
}

// Invariant: A[0...i-1] is sorted
function insertionSort(nums) {
  for (let i = 1; i < nums.length; i++) {
    let cur = nums[i],
      j = i;
    while (j > 0 && nums[j - 1] > cur) {
      nums[j] = nums[j - 1];
      j--;
    }
    nums[j] = cur;
  }
  return nums;
}

// Invariant: A[0...i-1] <= A[i...n-1] and A[0...i-1] is sorted
function selectionSort(nums) {
  for (let i = 0; i < nums.length - 1; i++) {
    let min = i;
    for (let j = i; j < nums.length; j++) {
      if (nums[j] < nums[min]) min = j;
    }
    [nums[min], nums[i]] = [nums[i], nums[min]];
  }
  return nums;
}

// Invariant: A[0...n-i-1] <= a[n-i...n-1], A[n-i...n-1] is sorted
function bubbleSort(nums) {
  for (let i = 0; i < nums.length - 1; i++) {
    for (let j = 0; j < nums.length - i - 1; j++) {
      if (nums[j] > nums[j + 1]) {
        [nums[j], nums[j + 1]] = [nums[j + 1], nums[j]];
      }
    }
  }
  return nums;
}
```
