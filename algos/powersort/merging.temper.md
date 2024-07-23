## Copy

TODO Builtin copy?

    let copy<T>(
      from: Listed<T>, begin: Int, end: Int, to: ListBuilder<T>
    ): Void {
      // TODO Assert size before starting?
      for (var i = begin; i < end; i += 1) {
        to[i - begin] = from[i];
      }
    }

    test("copy") { (test);;
      let random = new Random();

Make a random list and also a list of zeros with matching space available.

      let from = random.nextInts(100, 100);
      let to = new ListBuilder<Int>();
      fill(to, from.length, 0);

Copy a subrange.

      let begin = 10;
      let end = 20;
      copy(from, begin, end, to);

Make sure the copy matches and also that we don't just have zeros still. Zeros
might mean that we copied the wrong direction or something.

      assertIntsEqual(test, from.slice(begin, end), to.slice(0, end - begin));

And use a loop for this instead of reduce so be-java doesn't need to cope with
mixed up SAM types. TODO Fix be-java.

      let toAllZero = to.reduceFrom(true) { (all: Boolean, val): Boolean;;
        all && val == 0
      };
      assert(!toAllZero);
    }

## Extend and Reverse Run

Figure out if we can easily extract an already sorted prefix, even if we need to
reverse it in place.

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

Decreasing prefix is strict, presumably because we need to be stable, so we
don't want to change order for equal things. Merge will sort later, I reckon.

    test("extend decreasing prefix") { (test);;
      let ints = [5, 3, 2, 2, 4, 7, 6].toListBuilder();
      let check(prefixLength: Int, suffixLength: Int): Void {
        testAffixedInts(test, ints, prefixLength, suffixLength) { (full);;
          let intsEnd = prefixLength + ints.length;
          let end = extendAndReverseRunEnd(
            full, prefixLength, intsEnd, compareInts
          );
          assert(end - prefixLength == 3);
          let slice = full.slice(prefixLength, intsEnd);
          assertIntsEqual(test, slice, [2, 3, 5, 2, 4, 7, 6]);
        }
      }
      check(0, 0);
      check(5, 10);
    }

Increasing prefix is weak, presumably because we aren't changing order.

    test("extend increasing prefix") { (test);;
      let ints = [2, 2, 3, 5, 4, 7, 6];
      let check(prefixLength: Int, suffixLength: Int): Void {
        testAffixedInts(test, ints, prefixLength, suffixLength) { (full);;
          let intsEnd = prefixLength + ints.length;
          let end = extendAndReverseRunEnd(
            full, prefixLength, intsEnd, compareInts
          );
          assert(end - prefixLength == 4);
          assertIntsEqual(test, full.slice(prefixLength, intsEnd), ints);
        }
      }
      check(0, 0);
      check(5, 10);
    }

    let testAffixedInts(
      test: Test,
      ints: Listed<Int>,
      prefixLength: Int,
      suffixLength: Int,
      check: fn (ListBuilder<Int>): Void,
    ): Void {
      let random = new Random();
      let full = new ListBuilder<Int>();
      full.addAll(random.nextInts(prefixLength, 100));
      full.addAll(ints);
      full.addAll(random.nextInts(suffixLength, 100));
      check(full);
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

    test("merge few") { (test);;
      let ints = [3, 3, 5, 1, 4].toListBuilder();
      let buffer = new ListBuilder<Int>();
      fill(buffer, ints.length, 0);
      mergeRunsBasic(ints, 0, 3, ints.length, buffer, compareInts);

Conver to list because be-csharp IList vs IReadOnlyList. TODO Fix be-csharp.

      assertIntsEqual(test, ints.toList(), [1, 3, 3, 4, 5]);
    }

    test("merge many") { (test);;
      testMergeMany(test, 0, 0);
      testMergeMany(test, 20, 10);
    }

### Helper for Merge Sort Testing

    let testMergeMany(test: Test, prefixLength: Int, suffixLength: Int): Void {

Make random sorted runs.

      let random = new Random();
      let a = random.nextInts(50, 100).toListBuilder();
      let b = random.nextInts(20, 100).toListBuilder();
      insertionSort(a, 0, a.length, 0, compareInts);
      insertionSort(b, 0, b.length, 0, compareInts);

Put them in one list with unsorted prefix and suffix.

      let ints = new ListBuilder<Int>();
      let runsBegin = prefixLength;
      ints.addAll(random.nextInts(runsBegin, 100));
      ints.addAll(a);
      let runsMid = ints.length;
      ints.addAll(b);
      let runsEnd = ints.length;
      ints.addAll(random.nextInts(suffixLength, 100));
      assert(runsMid - runsBegin == a.length);
      assert(runsEnd - runsBegin == a.length + b.length);

Merge together.

      let buffer = new ListBuilder<Int>();
      fill(buffer, ints.length, 0);
      mergeRunsBasic(ints, runsBegin, runsMid, runsEnd, buffer, compareInts);
      assertSorted(test, ints.slice(runsBegin, runsEnd), compareInts);
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
