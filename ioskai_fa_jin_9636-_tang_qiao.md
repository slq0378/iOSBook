# iOS开发进阶-唐巧

- ARC使用
- `CoreFoundation`中不支持ARC，需要手动管理内存。
- 转换方式
    - `__bridge` 自己管理内存，不传递引用计数
    - `__bridge_retainded` 引用计数加1，但是释放还得要源对象
    - `__bridge_transfer` 传递引用计数，使用ARC管理内存
- GCD(Grand Central Dispatch)