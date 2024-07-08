## Fuzz Testing

Verify that we can sort longer lists correctly and stably.

    test("fuzz sorting") {
      let entries = new ListBuilder<Entry>();
      let random = new Random();
      for (var id = 0; id < 50; id += 1) {

Always ascend id, but randomly mix up a few categories. I tried just 2, but
apparently the prng is weak and was just alternating them, which was sad.

        entries.add({ id, category: random.nextInt(3) });
      }

If we sort by category, then `id` should still be ascending within each group.

      powersort(entries) { (a: Entry, b: Entry);; a.category - b.category };
      assert(false);
      // assert(entries.join("\n") { (a);; a.toString() } == "");
    }

We want a type with secondary data to make sure the order hasn't changed.

    class Entry {
      public id: Int;
      public category: Int;

      public toString(): String {
        "Entry { id: ${id.toString()}, category: ${category.toString()} }"
      }
    }
