## Insertion Sort

This is just a simple insertion sort on a given range of a list. It's used for
sorting sublists within powersort.

    let insertionSort<T>(
      items: ListBuilder<T>,
      begin: Int,
      end: Int,
      beginUnsorted: Int,
      compare: fn (T, T): Int,
    ): Void {
      // TODO Assert begin <= beginUnsorted && begin <= end?
      for (var i = beginUnsorted; i < end; i += 1) {
        var j = i;
        const v = items[i];
        while (j > begin && compare(v, items[j - 1]) < 0) {
          items[j] = items[j - 1];
          j -= 1;
        }
        items[j] = v;
      }
    }

    test("insertion sort with sorted prefix") { (test);;
      let ints = new ListBuilder<Int>();
      let random = new Random();

For testing, include an already sorted prefix.

      let sortedCount = 10;
      for (var i = 0; i < sortedCount; i += 1) {
        ints.add(i);
      }

Then include additional random values.

      ints.addAll(random.nextInts(100, 100));
      insertionSort(ints, 0, ints.length, sortedCount, compareInts);
      assertSorted(test, ints, compareInts);
    }

    test("insertion sort from the start") { (test);;
      let ints = new ListBuilder<Int>();
      let random = new Random();
      ints.addAll(random.nextInts(100, 100));
      insertionSort(ints, 0, ints.length, 0, compareInts);
      assertSorted(test, ints, compareInts);
    }

## Assert Sorted

    let assertSorted<T>(
      test: Test, items: Listed<T>, compare: fn (T, T): Int
    ): Void {
      var unordered = 0;
      for (var i = 1; i < items.length; i += 1) {
        if (compare(items[i - 1], items[i]) > 0) {
          unordered += 1;
        }
      }
      assert(unordered == 0);
    }

    let compareInts(a: Int, b: Int): Int { a - b }
