# 下拉刷新-上拉刷新

- tableView的上拉和下拉刷新可以通过

```objc
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    float height = scrollView.contentSize.height > _tableView.frame.size.height ?_tableView.frame.size.height : scrollView.contentSize.height;
    if ((height - scrollView.contentSize.height + scrollView.contentOffset.y) / height > 0.03) {
        // 上拉刷新
        [self loadData];
    }
    if (- scrollView.contentOffset.y / _tableView.frame.size.height > 0.2) {
        // 下拉刷新
    }
}
```