# 打开摄像机和相册

```objc
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    if (indexPath.row == 0) {
         // 弹出选择头像界面
        UIActionSheet *sheet = [[UIActionSheet alloc] initWithTitle:@"更换头像" delegate:self cancelButtonTitle:@"取消" destructiveButtonTitle:nil otherButtonTitles:@"从手机照片库选择",@"拍照", nil];
        [sheet showInView:self.view];
    }

- (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex
{
    switch (buttonIndex) {
        case  0: // 打开相册
        {
            UIImagePickerController *pickerVC = [[UIImagePickerController alloc] init];
            
            if ([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypePhotoLibrary]) {
                pickerVC.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
                pickerVC.allowsEditing = YES;
                pickerVC.delegate = self;
                [self presentViewController:pickerVC animated:YES completion:nil];
            }
        }
            break;
        case  1: // 打开摄像机
        {
            UIImagePickerController *pickerVC = [[UIImagePickerController alloc] init];
            
            if ([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
                pickerVC.sourceType = UIImagePickerControllerSourceTypeCamera;
                pickerVC.allowsEditing = YES;
                pickerVC.delegate = self;
                [self presentViewController:pickerVC animated:YES completion:nil];
            }
        }
            break;
        case  2:
        {
               NSLog(@"2");
        }
            break;
        default:
            break;
    }
}
#pragma mark - UIImagePickerControllerDelegate
/// 拍照相关
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info{
    
    UIImage *photo = info[UIImagePickerControllerOriginalImage];
    self.userImage = photo;
    [CommenDefault uploadUserImageWithUserId:CommonUserDef.userInfo.userId addImage:photo];
    [self.mainTable reloadData];
    
    [picker dismissViewControllerAnimated:YES completion:nil];
}

- (void)imagePickerControllerDidCancel:(UIImagePickerController *)picker{
    [picker dismissViewControllerAnimated:YES completion:nil];
}

```


- 封装的类方法

```objc
+ (void)uploadUserImageWithUserId:(NSString *)otherUserId addImage:(UIImage *)image
{
    NSString *userId = CommonUserDef.userInfo.userId;
    NSString *userPsw = [[MD5 MD5Encrypt:CommonUserDef.userInfo.password] uppercaseString];
    NSString *uid = otherUserId;
    
    NSMutableDictionary *para = [[NSMutableDictionary alloc] init];
    para[@"userId"] = userId;
    para[@"userPsw"] = userPsw;
    para[@"uid"] = uid;
    
    // 拼接图片名称
    NSString *imageName = [NSString stringWithFormat:@"%@%@",otherUserId,userPsw];
    // 截图
    NSData *shotImageData = UIImageJPEGRepresentation(image, 0.1);
    [AZFunction saveImage:shotImageData withidStr:imageName];
    
    NSString *url = [NSString stringWithFormat:@"https://%@:82/frame/webInteface.do?method=upHeadPic",NHBaseURL];
    NSLog(@"url:%@",url);
    [[AZHTTPClient shareInstance] POST:url parameters:para constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {

        NSData *data = [AZFunction getImageWithIdStr:imageName];
        [formData appendPartWithFileData:data name:@"headpic" fileName:@"headpic" mimeType:@"image/jpg"];
        
    } success:^(AFHTTPRequestOperation *operation, id responseObject) {
        if ([responseObject[@"result"] integerValue] == 0) {
            NSLog(@"上传头像失败");
            [MBProgressHUD showError:responseObject[@"errorMsg"]];
        }
        else
        {
            NSLog(@"上传头像成功，responseObject = %@",responseObject);
            [MBProgressHUD showSuccess:@"上传成功"];
        }
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"上传头像失败-%@",error);
        [MBProgressHUD showError:@"上传失败"];
    }];
}

+ (UIImage *)getUserPortraitWithUserId:(NSString *)otherUserId
{
    // 从本地读取图片，如果不存在就下载，否则就直接返回图片
    NSString *userId = CommonUserDef.userInfo.userId;
    NSString *userPsw = [[MD5 MD5Encrypt:CommonUserDef.userInfo.password] uppercaseString];
    NSString *uid = otherUserId;
    NSString *headtype = @"1";
    // 拼接图片名称
    NSString *imageName = [NSString stringWithFormat:@"%@%@",otherUserId,userPsw];
    // 从本地读取
    __block NSData *imageData = [AZFunction getImageWithIdStr:imageName];
    
    if (imageData == nil) { // 从网络下载
        NSMutableDictionary *para = [[NSMutableDictionary alloc] init];
        para[@"userId"] = userId;
        para[@"userPsw"] = userPsw;
        para[@"uid"] = uid;
        para[@"headtype"] = headtype;
        
        NSString *url = [NSString stringWithFormat:@"https://%@:82/frame/webInteface.do?method=downHeadPic",NHBaseURL];
        
        [[AZHTTPClient shareInstance] GET:url parameters:para success:^(AFHTTPRequestOperation *operation, id responseObject) {
            if (responseObject == nil) {
                if (operation.responseData) {
                    // 保存到本地
                    [AZFunction saveImage:operation.responseData withidStr:imageName];
                    imageData = operation.responseData;
                    NSLog(@"网络下载头像");
                }
            }
            else if ([responseObject[@"result"] integerValue] == 0) {
                NSLog(@"下载头像失败-%@",responseObject[@"errorMsg"]);
                [MBProgressHUD showError:responseObject[@"errorMsg"]];
            }
        } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            NSLog(@"下载头像失败-%@",error);
            [MBProgressHUD showError:@"下载失败"];
        }];
    }
    
    return [UIImage imageWithData:imageData];
}
```