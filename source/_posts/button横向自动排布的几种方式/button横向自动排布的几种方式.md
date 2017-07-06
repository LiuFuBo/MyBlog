---
title: button横向自动排布的几种方式
date: 2017-07-06 18:44:13
tags:
---

### 第一种方式:根据屏幕的宽度，然后计算一排能够放下多少个Button然后再来排布

<!-- more -->

	{for (int i=0; i<_dataSources.count; i++) {

	NSInteger rowCount = 0;

	if (SCREENWIDTH >320) {

	rowCount =(SCREENWIDTH - 46)/90 + 1;

	}else{

	rowCount =(SCREENWIDTH - 46)/90;

	}

	UIButton *perButton = [[UIButton alloc]init];

	perButton.tag = i+100;

	[perButton setTitle:_dataSources[i] forState:UIControlStateNormal];

	perButton.backgroundColor = NM_COLOR_RGB(244, 155, 56, 1);

	[perButton setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];

	perButton.layer.cornerRadius = 5;

	perButton.layer.masksToBounds = YES;

	[self addSubview:perButton];

	if (i<=(rowCount -1)) {

	[perButton mas_makeConstraints:^(MASConstraintMaker *make) {

	make.top.equalTo(self.timelimitLabel.mas_bottom).with.offset(23);

	if (i== 0) {

	make.left.equalTo(self.mas_left).with.offset(23);

	}else if(i> 0 && i<= rowCount - 1){

	make.left.mas_equalTo(i*93 + 23);

	}

	make.width.mas_equalTo(70);

	make.height.mas_equalTo(34);

	}];

	if (i == rowCount - 1 ) {

	_signCount = i;

	}

	}else if(i>(rowCount-1)){

	UIButton *topButton = (UIButton *)[self viewWithTag:_signCount+100];

	[perButton mas_makeConstraints:^(MASConstraintMaker *make) {

	make.top.equalTo(topButton.mas_bottom).with.offset(15);

	if(i%rowCount == 0){

	make.left.equalTo(self.mas_left).with.offset(23);

	}

	else if(i%rowCount > 0){

	make.left.mas_equalTo((i%rowCount)*93 + 23);

	}

	make.width.mas_equalTo(70);

	make.height.mas_equalTo(32);

	}];

	if (SCREENWIDTH > 320 && i%rowCount == 3) {

	_signCount = i;

	}else if(SCREENWIDTH == 320 && i%rowCount == 2){

	_signCount = i;

	}

	}

	}
	}
	
### 第二种：确定好第一排放多少个，然后把距离两边的边路设置好，中间的间隙自动改变：

	{for (int i=0; i<_dataSources.count; i++) {

	NSInteger rowCount =3;

	UIButton *perButton = [[UIButton alloc]init];

	perButton.tag = i+100;

	[perButton setTitle:_dataSources[i] forState:UIControlStateNormal];

	perButton.backgroundColor = NM_COLOR_RGB(244, 155, 56, 1);

	[perButton setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];

	perButton.layer.cornerRadius = 5;

	perButton.layer.masksToBounds = YES;

	[self addSubview:perButton];

	if (i<=(rowCount -1)) {

	[perButton mas_makeConstraints:^(MASConstraintMaker *make) {

	make.top.equalTo(self.timelimitLabel.mas_bottom).with.offset(23);

	if (i== 0) {

	make.left.equalTo(self.mas_left).with.offset(23);//(/3/2)

	}else if(i == 1){

	make.left.equalTo(self.mas_left).with.offset(95 + (SCREENWIDTH - 262)/2);

	}else if(i == 2){

	make.right.equalTo(self.mas_right).with.offset(-23);

	}

	make.width.mas_equalTo(72);

	make.height.mas_equalTo(34);

	}];

	if (i == rowCount - 1 ) {

	_signCount = i;

	}

	}else if(i>(rowCount-1)){

	UIButton *topButton = (UIButton *)[self viewWithTag:_signCount+100];

	[perButton mas_makeConstraints:^(MASConstraintMaker *make) {

	make.top.equalTo(topButton.mas_bottom).with.offset(15);

	if(i%rowCount == 0){

	make.left.equalTo(self.mas_left).with.offset(23);

	}

	else if(i%rowCount == 1){

	make.left.equalTo(self.mas_left).with.offset(95 + (SCREENWIDTH - 262)/2);

	}else if(i%rowCount == 2){

	make.right.equalTo(self.mas_right).with.offset(-23);

	}

	make.width.mas_equalTo(72);

	make.height.mas_equalTo(32);

	}];

	if(i%rowCount == 2){

	_signCount = i;
	
	}

	}

	}
	}

### 第三种:确定好第一排能放多少个，然后设置没一个间隙的间距都相同:

	{for (int i=0; i<_dataSources.count; i++) {

	NSInteger rowCount =3;

	UIButton *perButton = [[UIButton alloc]init];

	perButton.tag = i+100;

	[perButton setTitle:_dataSources[i] forState:UIControlStateNormal];

	perButton.backgroundColor = NM_COLOR_RGB(244, 155, 56, 1);

	[perButton setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];

	perButton.layer.cornerRadius = 5;

	perButton.layer.masksToBounds = YES;

	[self addSubview:perButton];

	if (i<=(rowCount -1)) {

	[perButton mas_makeConstraints:^(MASConstraintMaker *make) {

	make.top.equalTo(self.timelimitLabel.mas_bottom).with.offset(23);

	if (i== 0) {

	make.left.equalTo(self.mas_left).with.offset((SCREENWIDTH - 216)/4);//(/3/2)

	}else if(i == 1){

	make.left.equalTo(self.mas_left).with.offset(72 + (SCREENWIDTH - 216)/2);

	}else if(i == 2){

	make.right.equalTo(self.mas_right).with.offset(-((SCREENWIDTH - 216)/4));

	}

	make.width.mas_equalTo(72);

	make.height.mas_equalTo(34);

	}];

	if (i == rowCount - 1 ) {

	_signCount = i;

	}

	}else if(i>(rowCount-1)){

	UIButton *topButton = (UIButton *)[self viewWithTag:_signCount+100];

	[perButton mas_makeConstraints:^(MASConstraintMaker *make) {

	make.top.equalTo(topButton.mas_bottom).with.offset(15);

	if(i%rowCount == 0){

	make.left.equalTo(self.mas_left).with.offset((SCREENWIDTH - 216)/4);                }

	else if(i%rowCount == 1){

	make.left.equalTo(self.mas_left).with.offset(72 + (SCREENWIDTH - 216)/2);

	}else if(i%rowCount == 2){

	make.right.equalTo(self.mas_right).with.offset(-((SCREENWIDTH - 216)/4));

	}

	make.width.mas_equalTo(72);

	make.height.mas_equalTo(32);

	}];

	if(i%rowCount == 2){

	_signCount = i;

	}

	}

	}

	}