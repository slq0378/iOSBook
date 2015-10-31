# svn冲突解决

- Xcode 工程文件打开不出来， cannot be opened because the project file cannot be parsed.
- svn更新代码后，打开xcode工程文件，会出现  xxx..xcodeproj  cannot be opened because the project file cannot be parsed.
- 因为.xcodeproj工程文件冲突了，然后还是会强制更新，内部文件出现了冲突，所以解析不了文件。会出现这样的冲突消息
- 复制类似一下信息查找

```
<<<<<<< .mine  
    9ADAAC6A15DCEF6A0019ACA8 .... in Resources */,  
=======  
    52FD7F3D15DCEAEF009E9322 ... in Resources */,  
>>>>>>> .r269  
```

- 解决方法：
    - 1.对.xcodeproj 文件右键，显示包内容
    - 2.双击打开 project.pbxproj 文件
    - 3.找到以上类似的冲突信息（可以用commad + f 搜索）
    - 4.删除 <<<<<<<,======,>>>>>>这些行
    - 5.保存，退出
    - 6.重新打开.xcodeproj文件即可