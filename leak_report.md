# Leak report
## Charlot Shaw
Valgrind results

```
==21020==
==21020== HEAP SUMMARY:
==21020==     in use at exit: 46 bytes in 6 blocks
==21020==   total heap usage: 7 allocs, 1 frees, 1,070 bytes allocated
==21020==
==21020== 46 bytes in 6 blocks are definitely lost in loss record 1 of 1
==21020==    at 0x483AB1A: calloc (vg_replace_malloc.c:762)
==21020==    by 0x4011F8: strip (check_whitespace.c:41)
==21020==    by 0x401264: is_clean (check_whitespace.c:62)
==21020==    by 0x4012EC: main (check_whitespace.c:87)
==21020==
==21020== LEAK SUMMARY:
==21020==    definitely lost: 46 bytes in 6 blocks
==21020==    indirectly lost: 0 bytes in 0 blocks
==21020==      possibly lost: 0 bytes in 0 blocks
==21020==    still reachable: 0 bytes in 0 blocks
==21020==         suppressed: 0 bytes in 0 blocks
==21020==
==21020== For lists of detected and suppressed errors, rerun with: -s
==21020== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)

```

# Obeservations
`cleaned` on line 62 in the original file is used to make a calculation, and could be freed afterwards, say on line 69.
It is the only leak detected by Valgrind

## Solution
At first I tried directly to free `cleaned`, and it worked for most cases. However, I would be warned whenever the function was called on the empty string.
I tried at first to check if the string was empty, but couldn't find a way that didn't add more leaks. I turned the problem around and solved it by
using `strdup` to make a duplicate of the empty string, reasoning this would use `malloc` or it's brethren under the hood. This meant all returns from
strip would be `malloc`ed strings, and could be `free`d with egalitarian impunity.
