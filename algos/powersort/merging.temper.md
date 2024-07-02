## Copy

TODO Builtin copy?

    let copy<T>(
      from: ListBuilder<T>, begin: Int, end: Int, to: ListBuilder<T>
    ): Void {
      // TODO Assert size before starting?
      for (var i = begin; i < end; i += 1) {
        to[i - begin] = from[i];
      }
    }

## Extend and Reverse Run

    let extendAndReverseRunEnd<T>(
      items: ListBuilder<T>, begin: Int, end: Int, compare: fn (T, T): Int
    ): Int {
      let j = begin;
      if (j == end) { return j; }
      if (j + 1 == end) { return j + 1; }
      if (compare(items[j], items[j + 1]) > 0) {
        let prefixEnd = strictlyDecreasingPrefixEnd(items, begin, end, compare);
        reverse(items, begin, prefixEnd);
        prefixEnd
      } else {
        weaklyIncreasingPrefixEnd(items, begin, end, compare)
      }
    }

## Merge

This is the default merge in the reference.

Merges runs begin..&lt;middle and middle..&lt;end in-place into begin..&lt;end
by copying both to `buffer` and merging back into `items`. The buffer must have
space for full range.

    let mergeRunsBasic<T>(
      items: ListBuilder<T>,
      begin: Int,
      middle: Int,
      end: Int,
      buffer: ListBuilder<T>,
      compare: fn (T, T): Int,
    ): Void {
      copy(items, begin, end, buffer);
      var i1 = 0;
      let end1 = middle - begin;
      var i2 = end1;
      let end2 = end - begin;
      var out = begin;

Interleave based on compare ordering.

      while (i1 < end1 && i2 < end2) {
        items[out++] = if (compare(buffer[i1], buffer[i2]) <= 0) {
          // Post increment is much simpler in this case.
          buffer[i1++]
        } else {
          buffer[i2++]
        };
      }

Copy over remaining items. Only one of the two has any left, but it's simpler
just to make a loop of both.

      while (i1 < end1) {
        items[out++] = buffer[i1++];
      }
      while (i2 < end2) {
        items[out++] = buffer[i2++];
      }
    }

## Reverse

Our builtin reverse isn't on a subrange, so we need to implement that here.

    let reverse<T>(items: ListBuilder<T>, begin: Int, end: Int): Void {
      let mid = (end - begin) / 2;
      for (var i = 0; i < mid; i += 1) {
        let temp = items[begin + i];
        items[begin + i] = items[end - i - 1];
        items[end - i - 1] = temp;
      }
    }

Test both even and odd length subranges.

    test("reverse") { (test);;
      let numbers = [1, 2, 3, 4, 5].toListBuilder();
      reverse(numbers, 0, 3);
      reverse(numbers, 1, 5);
      assertIntsEqual(test, numbers.toList(), [3, 5, 4, 1, 2]);
    }

## Test Helpers

    let assertIntsEqual(
      test: Test, found: Listed<Int>, expected: Listed<Int>
    ): Void {
      assert(stringifyInts(found) == stringifyInts(expected));
    }

    let stringifyInts(ints: Listed<Int>): String {
      ints.join(", ") { (it);; it.toString() }
    }

## Prefixes and Suffixes

    let weaklyIncreasingPrefixEnd<T>(
      items: ListBuilder<T>, var begin: Int, end: Int, compare: fn (T, T): Int
    ): Int {
      while (begin + 1 < end && compare(items[begin], items[begin + 1]) <= 0) {
        begin += 1;
      }
      begin + 1
    }

    let strictlyDecreasingPrefixEnd<T>(
      items: ListBuilder<T>, var begin: Int, end: Int, compare: fn (T, T): Int
    ): Int {
      while (begin + 1 < end && compare(items[begin], items[begin + 1]) > 0) {
        begin += 1;
      }
      begin + 1
    }
