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
        while (compare(v, items[j - 1]) < 0) {
          items[j] = items[j - 1];
          j -= 1;
          if (j <= begin) { break; }
        }
        items[j] = v;
      }
    }
