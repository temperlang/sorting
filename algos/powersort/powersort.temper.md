# Powersort

Powersort is the [new default sorting algorithm in Python 3.11][python311]. It's
a stable sorting algorithm with some good properties. Code here is based on the
[reference implementation by Sebastian Wild][powersort].

    export class Powersort<T> {

## Compare

Parameterize the `compare` function at the instance level for convenience.

      public compare: fn (T, T): Int;

## Options

      public minRunLength: Int = 24;

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

      powerSortPaper(items: ListBuilder<T>, begin: Int, end: Int): Void {
        let length = end - begin;
        let maxStackHeight = floorLog2(length) + 1;

And in the reference implementation, they can allocate the stack array on the
call stack, but we can't do that here.

        let stack = new ListBuilder<RunBeginPower>();
        fill(stack, maxStackHeight, new RunBeginPower());
        var top = 0;

        let runA = new RunPower(
          begin, extendAndReverseRunEnd(items, begin, end, compare), 0
        );
        extendToMinRunLength(items, end, runA);
        while (runA.end < end) {
          let runB = new Run(
            runA.end, extendAndReverseRunEnd(items, runA.end, end, compare)
          );
          extendToMinRunLength(items, end, runB);
          runA.power = nodePowerDiv(
            0, length, runA.begin - begin, runB.begin - begin, runB.end - begin
          );

Invariant: Powers on stack must be increasing from bottom to top.

          while (stack[top].power > runA.power) {
            let topRun = stack[top];
            top -= 1;
            // merge_runs<mergingMethod>(top_run.begin, runA.begin, runA.end, _buffer.begin());
            runA.begin = topRun.begin;
          }

Store updated runA to be merged with runB at power k. And reuse objects to avoid
excess garbage.

TODO Value structs in Temper.

          top += 1;
          stack[top].begin = runA.begin;
          stack[top].power = runA.power;
          runA.begin = runB.begin;
          runA.end = runB.end;
          runA.power = 0;
        }
        // assert(runA.end == end);
        while (top > 0) {
          let topRun = stack[top];
          top -= 1;
          // merge_runs<mergingMethod>(top_run.begin, runA.begin, end, _buffer.begin());
          runA.begin = topRun.begin;
        }
      }

      extendToMinRunLength(items: ListBuilder<T>, end: Int, run: Runny): Void {
        let length = run.length;
        if (length < minRunLength) {
          run.end = end.min(run.begin + minRunLength);
          let beginUnsorted = run.begin + length;
          insertionSort(items, run.begin, run.end, beginUnsorted, compare);
        }
      }

The reference has multiple implementations for node power where the default has
no looping at all, but it wouldn't be easy to implement in Temper today. This
"div" version seems the most straightforward. I'm not sure if it effects the big
O.

      nodePowerDiv(
        begin: Int, end: Int, beginA: Int, beginB: Int, endB: Int
      ): Int {
        let twoN = 2 * (end - begin);
        let n1 = beginB - beginA;
        let n2 = endB - beginB;
        var a = 2 * beginA + n1 - 2 * begin;
        var b = 2 * beginB + n2 - 2 * begin;
        var k = 0;
        while (b - a <= twoN && a / twoN == b / twoN) {
          a *= 2;
          b *= 2;
          k += 1;
        }
        k
      }

      let buffer: ListBuilder<T> = new ListBuilder();
    }

## Run Types

As in, these relate to runs or sequences of items. We don't typically control
whether immutable slices of immutable lists are full copies or not, so make a
custom type where we know we aren't making copies.

    interface Runny {
      public get begin(): Int;
      public get end(): Int;
      public set end(value: Int): Void;
      public get length(): Int { end - begin + 0 }
    }

    class Run extends Runny {
      // public items: Listed<T>;
      public begin: Int = 0;
      public var end: Int = 0; // items.length;
    }

    class RunPower extends Runny {
      // public items: Listed<T>;
      public var begin: Int = 0;
      public var end: Int = 0;
      public var power: Int = 0;
    }

    class RunBeginPower {
      // public items: Listed<T>;
      public var begin: Int = 0;
      public var power: Int = 0;
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

    test("sort") { (test);;
      let ints = [4, 5, 1, 2, 3].toListBuilder();
      let sorted = [1, 2, 3, 4, 5];
      new Powersort(
        compare = fn (a: Int, b: Int) { a - b },

Keep `minRunLength` short for testing so we don't `insertionSort` everything.

        minRunLength = 3,
      ).sort(ints);

Needs `.toList()` because of C\# IList vs IReadOnlyList.

TODO Fix the need for this in be-csharp.

      assertIntsEqual(test, ints.toList(), sorted);
    }

[powersort]: https://github.com/sebawild/powersort/blob/48e31e909280ca43bb2c33dd3df9922b0a0f3f84/src/sorts/powersort.h
[python311]: https://docs.python.org/release/3.11.0/whatsnew/changelog.html
