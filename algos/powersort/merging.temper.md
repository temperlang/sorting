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
