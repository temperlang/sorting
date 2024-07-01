    class Powersort<T> {

      public compare: fn (T, T): Int;

      public sort(items: ListBuilder<T>): Void {
        //
      }

    }


    test("sort") {
      let ints = [4, 5, 1, 2, 3].toListBuilder();
      let sorted = [1, 2, 3, 4, 5];
      new Powersort(fn (a: Int, b: Int): Int { a - b }).sort(ints);
      for (var i = 0; i < ints.length; i += 1) {
        assert(ints[i] == sorted[i]);
      }
    }
