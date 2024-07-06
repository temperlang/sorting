TODO Fuzz testing

## Simple PRNG

We need some kind of interesting data for fuzz testing, so here's some pseudo
random number generation.

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
    }

    let randomMod = 0x7FFF_FFFF;

### Test Random

    test("random") { (test);;
      let random = new Random();
      let ints = new ListBuilder<Int>();
      for (var i = 0; i < 5; i += 1) {
        ints.add(random.nextInt(100));
      }

If we get enough bits, these ought to be consistent.

      assertIntsEqual(test, ints.toList(), [10, 77, 24, 83, 2]);
    }

### Test Random Parts

    test("temporary random debugging") {
      let seed = 7331;
      assert(new Random().next() == 331635110);


> local t = require("temper-core"); local r = t.band((1664525.0 * 7331) + 1.013904223E9, 2.147483647E9); print(r)
8921569702.0
Lua 5.4.2  Copyright (C) 1994-2020 Lua.org, PUC-Rio
> (1664525.0 * 7331 + 1.013904223E9) & 2.147483647E9
331635110
LuaJIT 2.1.1720049189 -- Copyright (C) 2005-2023 Mike Pall. https://luajit.org/
JIT: ON SSE3 SSE4.1 BMI2 fold cse dce fwd dse narrow loop abc sink fuse
> print(bit.band(1664525.0 * 7331 + 1.013904223E9, 2.147483647E9))
331635110
Lua 5.2.4  Copyright (C) 1994-2015 Lua.org, PUC-Rio
> print(bit32.band(1664525.0 * 7331 + 1.013904223E9, 2.147483647E9))
331635110

    }
