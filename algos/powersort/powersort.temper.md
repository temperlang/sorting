# Powersort

Powersort is the [new default sorting algorithm in Python 3.11][python311]. It's
a stable sorting algorithm with some good properties. Code here is based on the
[reference implementation by Sebastian Wild][powersort].

    class Powersort<T> {

## Compare

Parameterize the `compare` function at the instance level for convenience.

      public compare: fn (T, T): Int;

## Sort

We effectively sort in place, although using a separate buffer. Timsort often
can get by on less buffer size than the list being sorted, but at least in the
reference implementation, powersort needs more.

      public sort(items: ListBuilder<T>): Void {

Just short circuit if we have a short list.

        if (items.length < 2) {
          return;
        }

So we now know we have at least one value we can bad the buffer with. They want
items.length + 2 in the reference implementation.

        fill(buffer, items.length + 2, items[0]);
        powerSortPaper(items, 0, items.length);
      }

## Power Sort Paper

The reference implementation defaults `usePowerIndexedStack` to `false`, which
results in using the paper version of the algorithm. The relevant comment says,
"no measurable difference".

Meanwhile, code comments also say:

> sorts [begin,end), assuming that [begin,leftRunEnd) and * [rightRunBegin,end)
> are sorted

      powerSortPaper(items: Listed<T>, begin: Int, end: Int): Void {
        let length = end - begin;
        let maxStackHeight = floorLog2(length) + 1;

And in the reference implementation, they can allocate the stack array on the
call stack, but we can't do that here.

        let stack = new ListBuilder<RunBeginNPower>();
        fill(stack, maxStackHeight, new RunBeginNPower());
      }

      let buffer: ListBuilder<T> = new ListBuilder();
    }

## Run Types

As in, these relate to runs or sequences of items.

    class RunBeginNPower {
      //
    }

## Support

### Fill

TODO Clear and fill methods on ListBuilder.

    let fill<T>(list: ListBuilder<T>, minLength: Int, item: T): Void {
      while (list.length < minLength) {
        list.add(item);
      }
    }

### Floor Log2

Maybe a de Bruijn sequence would be much faster than this if we don't have int
primitive operations for it.

TODO Faster int log2 calculations!

    let log2 = 2.0.log();

    let floorLog2(n: Int): Int {
      (n.toFloat64().log() / log2).floor().toInt()
    }

## Tests

    test("sort") {
      let ints = [4, 5, 1, 2, 3].toListBuilder();
      let sorted = [1, 2, 3, 4, 5];
      new Powersort(fn (a: Int, b: Int) { a - b }).sort(ints);
      for (var i = 0; i < ints.length; i += 1) {
        assert(ints[i] == sorted[i]);
      }
    }

[powersort]: https://github.com/sebawild/powersort/blob/48e31e909280ca43bb2c33dd3df9922b0a0f3f84/src/sorts/powersort.h
[python311]: https://docs.python.org/release/3.11.0/whatsnew/changelog.html
