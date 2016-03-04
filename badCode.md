# 我从烂代码中学会了什么

### 目录
   - [不要再面向对象开发了] (#不要再面向对象开发了)
   - [谁的工作谁干] (#谁的工作谁干)
   - [第三方库入侵太深] (#第三方库入侵太深)
   - [用原生控件] (#用原生控件)
   - [App的主色调] (#App的主色调)
   
  
### 不要再面向对象开发了
现在移动开发机会是零门槛。注意机会是零门槛，不是没门槛。如果你是个移动开发者如果看到这个样的代码会怎么样？
```OC
            _idea = [responseObject objectForKey:@"idea"];
            _isOwner = [[responseObject objectForKey:@"is_owner"] boolValue];
            _hasFollowed = [[responseObject objectForKey:@"has_followed"] boolValue];
            _commentsCount = [[[responseObject objectForKey:@"idea"] objectForKey:@"comment_count"] integerValue];
            NSDictionary *owner = [_idea objectForKey:@"owner"];
            NSURL *coverUrl = [NSURL URLWithString:[_idea valueForKey:@"medium_cover_url"]];
```
面向字典开始真的很痛苦，面试模型开发吧。提高代码的可读性和易维护性。在字典转模型的时可以使用明杰老师的[MJExtension](https://github.com/CoderMJLee/MJExtension)会大大加快效率。

### 谁的工作谁干
不要有一万行代码都写在ViewController里，谁的工作谁干。搞个MVC。
```OC
   AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
    NSDictionary *owner = [_idea objectForKey:@"owner"];
    if(indexPath.row == 0){
        IdeaProfileTableViewCell *cell = [[IdeaProfileTableViewCell alloc] init];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        NSURL *coverUrl = [NSURL URLWithString:[_idea valueForKey:@"medium_cover_url"]];
        [cell.cover sd_setImageWithURL:coverUrl placeholderImage:[UIImage imageNamed:@"medium_idea_cover"]];
        cell.cover.userInteractionEnabled = YES;
        UITapGestureRecognizer *showCover = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(zoom:)];
        [cell.cover addGestureRecognizer:showCover];
        
        cell.title.text = [NSString stringWithFormat:@"%@", [_idea objectForKey:@"title"]];
        [cell.title sizeToFit];
        if (cell.title.frame.size.width + 20> appDelegate.rootController.view.frame.size.width-140-25) {
            cell.title.frame = CGRectMake(140, 20, appDelegate.rootController.view.frame.size.width-140-25 -20, 40);
            [cell.qrCode setFrame:CGRectMake(cell.title.frame.origin.x +cell.title.frame.size.width, cell.title.frame.origin.y+2, 16, 16)];
        }else {
            cell.title.frame = CGRectMake(cell.title.frame.origin.x, cell.title.frame.origin.y, cell.title.frame.size.width + 20, cell.title.frame.size.height+2);
            [cell.qrCode setFrame:CGRectMake(cell.title.frame.origin.x +cell.title.frame.size.width, cell.title.frame.origin.y+2, 16, 16)];
        }
        [cell.qrCode setUserInteractionEnabled:YES];
        UITapGestureRecognizer *showQRCode = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(showQR:)];
        [cell.qrCode addGestureRecognizer:showQRCode];
        
        NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
        [dateFormatter setDateFormat:@"yyyy-MM-dd'T'HH:mm:ssZ"];
        NSDate *date = [dateFormatter dateFromString:[_idea objectForKey:@"created_at"]];
        cell.timeAgo.text = [NSString stringWithFormat:@"%@ 发布", [date timeAgo]];
        
        if ([owner objectForKey:@"name"]) {
            NSMutableAttributedString *founder = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"创始人：%@", [owner objectForKey:@"name"]]];
            [founder addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 3)];
            [founder addAttribute:NSForegroundColorAttributeName value:TableColor range:NSMakeRange(4, [founder length] - 4)];
            cell.founder.attributedText = founder;
            cell.founder.userInteractionEnabled = YES;
            UserDataTapGestureRecognizer *showFounder = [[UserDataTapGestureRecognizer alloc] initWithTarget:self action:@selector(showOwner:)];
            showFounder.userData = [[owner objectForKey:@"server_id"] stringValue];
            showFounder.numberOfTapsRequired = 1;
            [cell.founder addGestureRecognizer:showFounder];
        }
        
        if ([[owner objectForKey:@"roles"] count] > 1) {
            NSMutableAttributedString *roles = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"角色定位：%@等", [[owner objectForKey:@"roles"] firstObject]]];
            [roles addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
            [roles addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [roles length] - 5)];
            cell.role.attributedText = roles;
        }else if ([[owner objectForKey:@"roles"] count] == 1) {
            NSMutableAttributedString *roles = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"角色定位：%@", [[owner objectForKey:@"roles"] firstObject]]];
            [roles addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
            [roles addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [roles length] - 5)];
            cell.role.attributedText = roles;
        }
        else if ([[owner objectForKey:@"roles"] count] == 0) {
            NSMutableAttributedString *roles = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"角色定位：未填写"]];
            [roles addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
            [roles addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [roles length] - 5)];
            cell.role.attributedText = roles;
        }
        
        if ([[_idea objectForKey:@"investment"] isEqual:[NSNull null]]) {
            NSMutableAttributedString *investment = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"可投入资金：未填写"]];
            [investment addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 5)];
            [investment addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(6, [investment length] - 6)];
            cell.money.attributedText = investment;
        }else if ([_idea objectForKey:@"investment"]) {
            NSMutableAttributedString *investment = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"可投入资金：%@", [_idea objectForKey:@"investment"]]];
            [investment addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 5)];
            [investment addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(6, [investment length] - 6)];
            cell.money.attributedText = investment;
        }
        
        if ([[owner objectForKey:@"experience"] isEqual:[NSNull null]]) {
            NSMutableAttributedString *experience = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"行业经验：未填写"]];
            [experience addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
            [experience addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [experience length] - 5)];
            cell.experience.attributedText = experience;
        }else if ([owner objectForKey:@"experience"]) {
            NSMutableAttributedString *experience = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"行业经验：%@", [owner objectForKey:@"experience"]]];
            [experience addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
            [experience addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [experience length] - 5)];
            cell.experience.attributedText = experience;
        }
        [cell.founder sizeToFit];
        [cell.role sizeToFit];
        [cell.money sizeToFit];
        cell.followed.userInteractionEnabled = YES;
        cell.followMount.userInteractionEnabled = YES;
        cell.commentMount.userInteractionEnabled = YES;
        cell.comment.userInteractionEnabled = YES;
        UITapGestureRecognizer *showFollowers = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(viewFollowers:)];
        [cell.followed addGestureRecognizer:showFollowers];
        [cell.followMount addGestureRecognizer:showFollowers];
        UITapGestureRecognizer *showComments = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(pushToCommentView)];
        [cell.comment addGestureRecognizer:showComments];
        [cell.commentMount addGestureRecognizer:showComments];
        cell.followMount.text = [NSString stringWithFormat:@"%@", [_idea objectForKey:@"followers_count"]];
        cell.commentMount.text = [NSString stringWithFormat:@"%@", [_idea objectForKey:@"comment_count"]];
        if ([[_idea objectForKey:@"recommend"] boolValue]) {
            cell.recommend.image = [UIImage imageNamed:@"ideaRecommend"];
        }
        else {
            cell.recommend.image = nil;
        }
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.backgroundColor = [UIColor colorWithHexString:@"F7F7F7"];
        return cell;
    }
```
很抱歉让大家看了一个这么长的代码。这是一个哥们的cell，我想告诉你这个TableView里边有10个自定义的cell，你可以想象.......此处省略10000字。用MVC模式之后
```OC
      if (indexPath.row == 0) {
        UserMainInfoCell *cell = [[UserMainInfoCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"cellOne"];
        cell.userInfoModel = self.userInfoModel;
        return cell;
    }
```
ViewController你把Model给我，剩下的事情和你没关系。

### 第三方库入侵太深
我举一个例子AFNetworking，这个可能是每一个开发者网络请求的标配，机会每一个页面都要用到。这样就造成了AFNetworking入侵太深，最近AFNetworking升级到了3.0，很多类改了方法也改了，你一看自己的项目300个地方用到了AFNetworking怎么办。全局替换？？像这样的库最好进行二次封装。
```OC
   -(void)GET:(NSString*)url withParameters:(NSDictionary*)parameters success:(void (^)(NSDictionary *data))success failure:(void (^)(NSError *error,AFHTTPRequestOperation *operation))failure
{
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
    [manager GET:url parameters:parameters success:^(AFHTTPRequestOperation *operation, id responseObject) {
        if (success)
        {
            success((NSDictionary*)responseObject);
        }
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        if (failure)
        {
            failure(error,operation);
        }
    }];
}
```
封装之后使用，如果AFNetworking更新了你就改此处即可。

### 用原生控件
如果没有特殊需求，尽量用苹果提高的原生控件。有个哥们的TabBarViewController不知道为啥要用。
```OC
   @interface RDVTabBarController : UIViewController <RDVTabBarDelegate>
```
没有特殊需求不要用第三方的东西，而且实现的功能是一样的。

### App的主色调开发
开发之前确立好App的主色调不要每次用到颜色就用这个东西
```OC
   self.view.backgroundColor = [UIColor colorWithHexString:@"878787"];
   titleLabel.backgroundColor = [UIColor colorWithRed:0.95f green:0.48f blue:0.00f alpha:1.00f];
```
确定颜色之后定义成宏。
```OC
   self.view.backgroundColor = DLBackgroundColor;
   titleLabel.backgroundColor = DLBackgroundColor;
``

