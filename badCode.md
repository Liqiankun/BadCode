# 我从烂代码中学会了什么

### 目录
   - [不要再面向对象开发了] (#不要再面向对象开发了)
   - [谁的工作谁干] (#谁的工作谁干)
  
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

