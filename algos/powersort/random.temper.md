## Simple PRNG

We need some kind of interesting data for fuzz testing, so here's some pseudo
random number generation.

TODO Move this to some separate project? Add prng to temper core?

    class Random {
      public var seed: Int = 7331;

Pretend we can get at least 32-bit signed ints, even though we're still not sure
we can promise that for all backends.

      public next(): Int {
        let a = 1_664_525;
        let c = 1_013_904_223;

Anding bits here ought to prevent negatives if 32 bits.

        seed = (a * seed + c) & randomMod;
        if (seed == randomMod) {

Rejection sampling at border. Probably something smarter to do but meh.

          next()
        } else {
          seed
        }
        return seed;
      }

Int in the range 0..&lt;end.

      public nextInt(end: Int): Int {
        next() % end
      }

      public nextInts(length: Int, valueEnd: Int): List<Int> {
        let ints = new ListBuilder<Int>();
        for (var i = 0; i < length; i += 1) {
          ints.add(nextInt(valueEnd));
        }
        ints.toList()
      }
    }

    let randomMod = 0x7FFF_FFFF;

## Random Int List

## Test Random

    test("random") { (test);;

If we get enough bits, these ought to be consistent.

      let ints = new Random().nextInts(3, 100);
      assertIntsEqual(test, ints.toList(), [10, 77, 24]);

Past the first 3, we get other results on some versions of Lua.

      // assertIntsEqual(test, ints.toList(), [10, 77, 24, 83, 2]);
      // On lua 5.2 or luajit, we get [10, 77, 24, 92, 75]
    }
