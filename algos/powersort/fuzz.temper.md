## Fuzz Testing

Verify that we can sort longer lists correctly and stably.

    test("fuzz sorting") { (test);;
      let entries = new ListBuilder<Entry>();
      let random = new Random();
      let categoryCount = 3;
      for (var id = 0; id < 100; id += 1) {

Always ascend id, but randomly mix up a few categories. I tried just 2, but
apparently the prng is weak and was just alternating them, which was sad.

        entries.add({ id, category: random.nextInt(categoryCount) });
      }

If we sort by category, then `id` should still be ascending within each group.

      let compareCategory(a: Entry, b: Entry): Int { a.category - b.category }
      let compareId(a: Entry, b: Entry): Int { a.id - b.id }
      powersort(entries, compareCategory)

First check that the categories are sorted but that the ids as a whole aren't.

      assertSorted(test, entries, compareCategory);
      assertUnsorted(test, entries, compareId);

Within each group, check for sorted ids.

      for (var category = 0; category < categoryCount; category += 1) {

Copy to non-var, so Java doesn't have trouble with non-final access from
closure. TODO Fix be-java.

        let category = category;
        let group = entries.filter { (a);; a.category == category };
        assertSorted(test, group, compareId);
      }
    }

We want a type with secondary data to make sure the order hasn't changed.

    class Entry {
      public id: Int;
      public category: Int;

      public toString(): String {
        "Entry { id: ${id.toString()}, category: ${category.toString()} }"
      }
    }
