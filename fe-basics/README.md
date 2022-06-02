```js
/**
 * stack: nodes to return to
 * cur: node to enter
 */
function postOrder(root) {
  let res = [],
    stack = [],
    cur = root;
  while (cur || stack.length) {
    if (cur) {
      stack.push([cur, false]);
      cur = cur.left;
    } else {
      const [node, done] = stack.pop();
      if (done) {
        res.push(node.val);
      } else {
        stack.push([node, true]);
        cur = node.right;
      }
    }
  }
  return res;
}

function inorder(root) {
  let res = [],
    stack = [],
    cur = root;
  while (cur || stack.length) {
    if (cur) {
      stack.push(cur);
      cur = cur.left;
    } else {
      const node = stack.pop();
      res.push(node.val);
      cur = node.right;
    }
  }
  return res;
}

function preorder(root) {
  let res = [],
    stack = [],
    cur = root;
  while (cur || stack.length) {
    if (cur) {
      res.push(cur.val);
      stack.push(cur);
      cur = cur.left;
    } else {
      cur = stack.pop().right;
    }
  }
  return res;
}
```

### Sorting

```js
/**
 * Hoare partition scheme
 * - Invariant: A[L...l-1] <= pivot, A[r+1...R] >= pivot
 * - Terminating Condition: l == r + 1 or l == r
 *     +----+----+        +-----+-----+-----+
 *     | <p | >p |   or   | <=p | ==p | >=p |    partition point: l or r+1
 *     +----+----+        +-----+-----+-----+
 *     r   l,r+1               l,r   r+1
 *          ^                   ^     ^
 *   EDGE CASE I: pivot on the left, loop terminates after first iteration
 *     +---+---+      +---+
 *     | p |   |  ... |   |    l will cause infinite recursion
 *     +---+---+      +---+
 *     l  r+1
 *
 *   EDGE CASE II: pivot on the right, loop terminates after first iteration
 *     +---+---+      +---+
 *     |   |   |  ... | p |    r+1 will cause infinite recursion
 *     +---+---+      +---+
 *                    l  r+1
 */
function quickSort(nums) {
  function partition(l, r) {
    const pivot = nums[l];
    // const pivot = nums[Math.floor((l + r) / 2)];
    // const pivot = nums[r];
    // const pivot = nums[Math.ceil((l + r) / 2)];
    l--;
    r++;
    while (true) {
      do {
        l++;
      } while (nums[l] < pivot);
      do {
        r--;
      } while (nums[r] > pivot);
      if (l >= r) return r + 1;
      // if (l >= r) return l;
      [nums[l], nums[r]] = [nums[r], nums[l]];
    }
  }
  function sort(l, r) {
    if (l < r) {
      const mid = partition(l, r);
      sort(l, mid - 1);
      sort(mid, r);
    }
  }
  sort(0, nums.length - 1);
  return nums;
}

/**
 * Lomuto partition scheme
 * - Invariant: A[l...i-1] < pivot, A[i...j-1] >= pivot
 * - Terminating condition: j == r
 */
function quickSort(nums) {
  function partition(l, r) {
    const pivot = nums[r];
    let i = l;
    for (let j = l; j < r; j++) {
      if (nums[j] < pivot) {
        [nums[j], nums[i]] = [nums[i], nums[j]];
        i++;
      }
    }
    [nums[i], nums[r]] = [nums[r], nums[i]];
    return i;
  }
  function sort(l, r) {
    if (l < r) {
      const mid = partition(l, r);
      sort(l, mid - 1);
      sort(mid + 1, r);
    }
  }
  sort(0, nums.length - 1);
  return nums;
}

// Out-of-place version
function quickSort(nums) {
  function merge(left, pivot, right, arr) {
    let i = 0;
    for (let n of left) arr[i++] = n;
    arr[i++] = pivot;
    for (let n of right) arr[i++] = n;
  }
  function sort(arr) {
    if (arr.length < 2) return;
    const pivot = arr[0],
      left = [],
      right = [];
    for (let i = 1; i < arr.length; i++) {
      if (arr[i] < pivot) {
        left.push(arr[i]);
      } else {
        right.push(arr[i]);
      }
    }
    sort(left);
    sort(right);
    merge(left, pivot, right, arr);
  }
  sort(nums);
  return nums;
}

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
