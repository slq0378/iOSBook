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