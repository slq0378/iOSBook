# svn冲突解决

- Xcode 工程文件打开不出来， cannot be opened because the project file cannot be parsed.
- svn更新代码后，打开xcode工程文件，会出现  xxx..xcodeproj  cannot be opened because the project file cannot be parsed.
- 因为.xcodeproj工程文件冲突了，然后还是会强制更新，内部文件出现了冲突，所以解析不了文件。会出现这样的冲突消息