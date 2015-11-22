# 一起来做webgame，《Javascript贪食蛇》

> 写于：2013-08-14

今天来个略有意思的，《贪食蛇》。这个估计没有人没玩过吧。它稍有点难度，不过仍然算是简单的游戏，实现代码也不多，下面就开始

## 思路

贪食蛇主要几个问题需要解决

- 转向，蛇身每个点在经过转向点的时候都要转向
- 吃，每吃掉一颗，蛇身就增加一个点
- 失败，撞墙或撞到蛇身上，都算失败

基本上，《贪食蛇》就难在这三个地方，这个顺序，难度从高到低，最简单的莫过去撞墙判断失败。最难的是转向，之后是吃。下面从最开始一步步的来解决这些问题。

## 一些变量

```
var mapItemX=60;  //游戏地图横向点数
var mapItemY=31;  //游戏地图纵向点数
var snakeLen=5;  //蛇的初始长度
var snakeMoveDirection='E';  //蛇的移动方向
var snakeStartPoints={'x':5,'y':15};  //蛇的起始位置
var snake=new Array();  //用于存放蛇身点的坐标
var corner=new Array();  //用于存放转角点坐标
var cornerNum=0;  //转角数
var timer,speed=100;  //移动计时器和初始移动速度
var timeiner,timeSecond=0,timeMinute=0,timestr=0;  //时间计时器，分，秒，总秒数
var mouseX,mouseY;  //老鼠位置（吃的）
var start=false;  //是否开始
```

## 初始化地图

```
function init(){
	var maps='';
	for(var i=0;i<mapItemY;i++){
		for(var j=0;j<mapItemX;j++){
			maps=maps+'<div id="mapItem_'+j+'_'+i+'" class="mapItem"></div>';
		}
	}
	$("#game_map").html(maps);  //放地图的容器
}
```

地图很简单，不过要注意，必须保证以第一行0，0开始，第二行0，1开始，以此类推，它是一个二维数组，这直接关系到定位，所以必须得保证这样的结构。

生成的每一个点，都有一个根据纵横坐标组成的ID，它将是控制这些点的必要东西

## 初始化蛇

```
function initSnake(){
	if(snakeMoveDirection=='E'){  //用于判断蛇的初始前进方向
		var j=snakeLen+snakeStartPoints.x;
		var _snakeLen=snakeLen;
		for(var i=snakeStartPoints.x;i<j;i++){
			if(i==snakeLen*2-1){
				$("#mapItem_"+i+"_"+snakeStartPoints.y).addClass('snakehead');
			}else{
				$("#mapItem_"+i+"_"+snakeStartPoints.y).addClass('snakebody');
			}
			_snakeLen--;
			snake[_snakeLen]=new Array();
			snake[_snakeLen]['x']=i;
			snake[_snakeLen]['y']=snakeStartPoints.y;
			snake[_snakeLen]['d']=snakeMoveDirection;
			
		}
	}
}
```

snakeMoveDirection，有个默认值，直接决定蛇的初始前进方向，这里只使用了一个方向，所以只写了一组，如果需要设定蛇的初始前方向，这里得把4个方向都补齐。

之后根据初始的蛇身长度和初始位置进行循环，将之前的点增加CSS样式，也就是变个颜色，蛇头用另一种颜色，这样好区分那里是头。每个点都要写进snake数组里，从大到小写，比如初始蛇长为5，那就是4到0，蛇头永远是0，这样会更容易操作。

## 初始化吃的

```
function initMouse(){
	var _x,_y;
	var _this=this;
	
	this.randoms=function(){
		_x=Math.floor(Math.random()*(mapItemX-1));
		_y=Math.floor(Math.random()*(mapItemY-1));
		_this.checkMouse();
	}
	this.checkMouse=function(){
		var _e=false;
		for(var i in snake){
			if(snake[i]['x']==_x&&snake[i]['y']==_y){
				_e=true;
				break;
			}
		}
		if(_e==true){
			_this.randoms();
		}else{
			mouseX=_x;
			mouseY=_y;
			$("#mapItem_"+mouseX+'_'+mouseY).addClass('mouse');
		}
	}
	this.randoms();
}
```

吃的，也就是在地图上随机出现一个点，所以直接使用Math.random()就可以了，因为是横纵坐标，所以得生成两个值，它的下限自然是0，上限自然是地图的大小，所以使用：Math.floor(Math.random()*(mapItemX-1));;

不过这里还有个问题，这样随机的点，有可能刚好在蛇身上，那样可就太糟糕了，所以蛇身所在的点都得排除。this.checkMouse()，就是用来做这件事，遍历蛇身的点，和生成的点坐标进行比较，如果刚好在它身上，再生成一次随机点。这里是递归，this.randoms()里调用this.checkMouse()，这样直到生成的点不在蛇身上为止。找到这样的点后，就给这个点增加一个CSS样式，改变它的颜色，这里用红色，表明是吃的。

好了，准备工作就绪了，下面就是移动

## 移动

```
function move(){
	var _d;
	$("#mapItem_"+snake[snakeLen-1]['x']+'_'+snake[snakeLen-1]['y']).removeClass('snakebody');
	for(var i=0;i<snakeLen;i++){
		if(i==0){
			$("#mapItem_"+snake[i]['x']+'_'+snake[i]['y']).removeClass('snakehead');
			$("#mapItem_"+snake[i]['x']+'_'+snake[i]['y']).addClass('snakebody');
		}
		var _d=corners(snake[i]['x'],snake[i]['y']);  //判断转角
		if(_d){
			snake[i]['d']=_d;
		}
		if(i==snakeLen-1&&corner.length>0){
			clearCorners(snake[i]['x'],snake[i]['y']);  //清除一个转角点
		}
		switch(snake[i]['d']){
			case 'E':
				if(i==0){
					eat(snake[i]['x']+1,snake[i]['y']);
					checkDie(snake[i]['x']+1,snake[i]['y']);
				}
				snake[i]['x']=snake[i]['x']+1;
				break;
			case 'S':
				if(i==0){
					eat(snake[i]['x'],snake[i]['y']+1);
					checkDie(snake[i]['x'],snake[i]['y']+1);
				}
				snake[i]['y']=snake[i]['y']+1;
				break;
			case 'W':
				if(i==0){
					eat(snake[i]['x']-1,snake[i]['y']);
					checkDie(snake[i]['x']-1,snake[i]['y']);
				}
				snake[i]['x']=snake[i]['x']-1;
				break;
			case 'N':
				if(i==0){
					eat(snake[i]['x'],snake[i]['y']-1);
					checkDie(snake[i]['x'],snake[i]['y']-1);
				}
				snake[i]['y']=snake[i]['y']-1;
				break;
		}
	}

	$("#mapItem_"+snake[0]['x']+'_'+snake[0]['y']).addClass('snakehead');
}
```

移动的代码稍多，主要是思路是：

- 每个蛇身点，都有三个值：1、所在横坐标。2、所在纵坐标。3、下一步移动方向。移动方向是个很重要的值，初始化蛇的时候就已经根据初始前进方向设定了
- 普通移动只需要将蛇头前进方向的下一个点记为新的蛇头，而蛇尾的点从蛇身上移除
- 转角移动，需考虑目标方向，corner里记录有转角的坐标和方向，那么，当蛇身上的点经过转角点的时候，根据它的方向重设蛇身点的方向，根据不同的方向增加或减少对应的坐标值。每个点经过它的时候，都会做这样的事情，所以整条蛇就转过去了。代码中的switch，就是来做这个事情的。
- 每次移动，都对第一个点做判断：1、是不是吃到了，eat()。2、是不是死掉了，checkDie()。它们都有两个参数，分别是目标点的坐标，所以，4个方向都调用了一次这两个方法。

## 判断转角

```
function corners(x,y){
	var _d='';
	for(var i in corner){
		if(corner[i]['x']==x&&corner[i]['y']==y){
			_d=corner[i]['d'];
		}
	}
	return _d;
}
```

主要是遍历转角数组，判断蛇身点，是不是经过它，如果是就返回这个转角点的方向，move()的时候，如果corners()有值，则更新经过它的蛇身点的方向。每个点都会经过它，都会改变方向，整个蛇也就改变了方向。

## 清队转角

```
function clearCorners(x,y){
	if(corner[0]['x']==x&&corner[0]['y']==y){
		corner.shift();
		cornerNum=corner.length;
	}
}
```

之所以要清除转角，是因为如果你不清除它，那当蛇再次经过它的时候，会自动转向的。这里主要是先进先出的原则，直接使用shift()就可以了，之后重计转角数组的长度。

## 判断失败

```
function checkDie(x,y){
	if(x<0||x>mapItemX-1||y<0||y>mapItemY-1){ //撞墙
		clearInterval(timer);
		clearInterval(timeiner);
		var _scores=scores();
		alert('你死了！总分：'+_scores.toFixed(2));
		location.reload();
		return false;
	}
	for(var i in snake){
		if(snake[i]['x']==x&&snake[i]['y']==y){  //撞到蛇身上
			clearInterval(timer);
			clearInterval(timeiner);
			var _scores=scores();
			alert('你死了！总分：'+_scores.toFixed(2));
			location.reload();
			return false;
		}
	}
}
```

撞墙最好懂了，也就是超出了地图的边。撞蛇身上，则需要遍历蛇身上的点，判断移动到的点是不是在这些点里，如果是，你死了。

死了，自然要停止计时，给出提示。这里还计算了一下分数，不过分数的计算方法很差劲，算了，不说了。

## 吃食

```
function eat(x,y){
	if(mouseX==x&&mouseY==y){  //判断吃
		var _snakeLast=snake[snakeLen-1];
		snake[snakeLen]=new Array();
		switch(_snakeLast['d']){
			case 'E':
				snake[snakeLen]['x']=_snakeLast['x']-1;
				snake[snakeLen]['y']=_snakeLast['y'];
				break;
			case 'S':
				snake[snakeLen]['x']=_snakeLast['x'];
				snake[snakeLen]['y']=_snakeLast['y']-1;
				break;
			case 'W':
				snake[snakeLen]['x']=_snakeLast['x']+1;
				snake[snakeLen]['y']=_snakeLast['y'];
				break;
			case 'N':
				snake[snakeLen]['x']=_snakeLast['x'];
				snake[snakeLen]['y']=_snakeLast['y']+1;
				break;
		}
		snake[snakeLen]['d']=_snakeLast['d'];
		snakeLen=snake.length;
		$("#snake_len").html(snakeLen);
		$("#mapItem_"+mouseX+"_"+mouseY).removeClass('mouse');
		initMouse();
		changeSpeed();
	}
}
```

蛇头碰到吃点时，最重的一点是在蛇尾增加一个点做蛇尾，但需要考虑的问题是，这个点应该在蛇尾的什么位置上。这里使用蛇尾点的方向，如果它的方向是E，那就在它的W上增加一个点，同理可得其它3个点时，增加点的位置。也就是代码中switch部分所做的事情。

新的蛇尾添加好后，就可以把吃的点移除，removeClass()，之后再生成一个新的吃点initMouse();

changeSpeed()，只是为了提高一些可玩性，满足条件时加快一点蛇的移动速度。这里就不提了。

## 控制

```
function keyDirection(e){
	var evt = e ||window.event; 
	var key=evt.which||evt.keyCode; 
	switch(key){
		case 37:
			snakeMoveDirection='W';
			break;
		case 38:
			snakeMoveDirection='N';
			break;
		case 39:
			snakeMoveDirection='E';
			break;
		case 40:
			snakeMoveDirection='S';
			break;
	}
	corner[cornerNum]=new Array();
	corner[cornerNum]['x']=snake[0]['x'];
	corner[cornerNum]['y']=snake[0]['y'];
	corner[cornerNum]['d']=snakeMoveDirection;
	cornerNum++;
}
```

很简单，获取按键值，判断方向，设定snakeMoveDirection，然后增加一个转角点。但这里会有一个问题，那就是，当连续两次按键的速度快过蛇的移动速度时，两个转角点会重合，这就会让整个转角代码失控，出现不能清除的情况，那当蛇再次经过这些点的时候，它会自动转向，所以，这里还得加强一下

```
var _snakeMoveDirection=snakeMoveDirection;  //先将方向附给_snakMoveDirection

this.createCorner=function(){
	corner[cornerNum]=new Array();
	corner[cornerNum]['x']=snake[0]['x'];
	corner[cornerNum]['y']=snake[0]['y'];
	corner[cornerNum]['d']=snakeMoveDirection;
	$("#mapItem_"+corner[cornerNum]['x']+'_'+corner[cornerNum]['y']).addClass('corner');
	cornerNum++;
}

if(cornerNum>0){  //判断转角数是不是大于0
	if(corner[cornerNum-1]['x']!=snake[0]['x']||corner[cornerNum-1]['y']!=snake[0]['y']){
		this.createCorner();
	}else{
		snakeMoveDirection=_snakeMoveDirection;
	}
}else{
	this.createCorner();
}
```

使用一个变量_snakeMoveDirection来保存之前的方向，目的是为了，当判断点重合的时候，再将方向改过来，不然蛇会不听话的。判断转角数大于0后，再判断一下目标点的坐标和前一个转角点的坐标，是不是重合，如果不重合，那么记录新点，重合就不记录点了，而是将之前的方向再附给snakeMoveDirection。这样就不会出现快速按键时的问题了。

为了避免按下蛇前进方向的方向时仍然记录转角点，这里还可以增加一个判断，如果目标方向和前进方向相同，则不做任何记录。完整的控制代码：

```
function keyDirection(e){
	var evt = e ||window.event; 
	var key=evt.which||evt.keyCode;
	var notdo=false; 
	var _snakeMoveDirection=snakeMoveDirection;
	switch(key){
		case 37:
			if(snakeMoveDirection=='W'){
				notdo=true;
			}else{
				snakeMoveDirection='W';
			}
			break;
		case 38:
			if(snakeMoveDirection=='N'){
				notdo=true;
			}else{
				snakeMoveDirection='N';
			}
			break;
		case 39:
			if(snakeMoveDirection=='E'){
				notdo=true;
			}else{
				snakeMoveDirection='E';
			}
			break;
		case 40:
			if(snakeMoveDirection=='S'){
				notdo=true;
			}else{
				snakeMoveDirection='S';
			}
			break;
	}
	

	this.createCorner=function(){
		corner[cornerNum]=new Array();
		corner[cornerNum]['x']=snake[0]['x'];
		corner[cornerNum]['y']=snake[0]['y'];
		corner[cornerNum]['d']=snakeMoveDirection;
		$("#mapItem_"+corner[cornerNum]['x']+'_'+corner[cornerNum]['y']).addClass('corner');
		cornerNum++;
	}

	if(notdo==false){
		if(cornerNum>0){
			if(corner[cornerNum-1]['x']!=snake[0]['x']||corner[cornerNum-1]['y']!=snake[0]['y']){
				this.createCorner();
			}else{
				snakeMoveDirection=_snakeMoveDirection;
			}
		}else{
			this.createCorner();
		}
	}
}
```

程序基本上就这些内容，后面的计数、计分、计时，就不说了。最后是使用定时器，让蛇移动起来

```
$("#game_map").click(function(){
	if(start==false){
		start=true;
		timeiner=setInterval(timeing,1000);
		move();
		timer=setInterval(move,speed);
	}else{
		start=false;
		clearInterval(timer);
		clearInterval(timeiner);
	}
});
```

这里使用的是点击游戏地图开始的方式，为了防止加速的情况，所以使用了start变量做了一个判断。点第一次是开始，再点就是暂停，再点又是开始。

最后是完整代码(JS部分，HTML请直接查看原代码)：

```
var mapItemX=60;
var mapItemY=31;
var snakeLen=5;
var snakeMoveDirection='E';
var snakeStartPoints={'x':5,'y':15};
var snake=new Array();
var corner=new Array();
var cornerNum=0;
var timer,speed=100;
var timeiner,timeSecond=0,timeMinute=0,timestr=0;
var mouseX,mouseY;
var start=false;

$("#game_map").click(function(){
	if(start==false){
		start=true;
		timeiner=setInterval(timeing,1000);
		move();
		timer=setInterval(move,speed);
	}else{
		start=false;
		clearInterval(timer);
		clearInterval(timeiner);
	}
});

$(document).keydown(function(e){
	keyDirection(e);
});

init();

function init(){
	
	var maps='';
	for(var i=0;i<mapItemY;i++){
		for(var j=0;j<mapItemX;j++){
			maps=maps+'<div id="mapItem_'+j+'_'+i+'" class="mapItem"></div>';
		}
	}
	$("#game_map").html(maps);
	initSnake();
	initMouse();
}

function initSnake(){
	if(snakeMoveDirection=='E'){
		var j=snakeLen+snakeStartPoints.x;
		var _snakeLen=snakeLen;
		for(var i=snakeStartPoints.x;i<j;i++){
			if(i==snakeLen*2-1){
				$("#mapItem_"+i+"_"+snakeStartPoints.y).addClass('snakehead');
			}else{
				$("#mapItem_"+i+"_"+snakeStartPoints.y).addClass('snakebody');
			}
			_snakeLen--;
			snake[_snakeLen]=new Array();
			snake[_snakeLen]['x']=i;
			snake[_snakeLen]['y']=snakeStartPoints.y;
			snake[_snakeLen]['d']=snakeMoveDirection;
			
		}
	}
}

function move(){
	var _d;
	$("#mapItem_"+snake[snakeLen-1]['x']+'_'+snake[snakeLen-1]['y']).removeClass('snakebody');
	for(var i=0;i<snakeLen;i++){
		if(i==0){
			$("#mapItem_"+snake[i]['x']+'_'+snake[i]['y']).removeClass('snakehead');
			$("#mapItem_"+snake[i]['x']+'_'+snake[i]['y']).addClass('snakebody');
		}
		var _d=corners(snake[i]['x'],snake[i]['y']);
		if(_d){
			snake[i]['d']=_d;
		}
		if(i==snakeLen-1&&corner.length>0){
			clearCorners(snake[i]['x'],snake[i]['y']);
		}
		switch(snake[i]['d']){
			case 'E':
				if(i==0){
					eat(snake[i]['x']+1,snake[i]['y']);
					checkDie(snake[i]['x']+1,snake[i]['y']);
				}
				snake[i]['x']=snake[i]['x']+1;
				break;
			case 'S':
				if(i==0){
					eat(snake[i]['x'],snake[i]['y']+1);
					checkDie(snake[i]['x'],snake[i]['y']+1);
				}
				snake[i]['y']=snake[i]['y']+1;
				break;
			case 'W':
				if(i==0){
					eat(snake[i]['x']-1,snake[i]['y']);
					checkDie(snake[i]['x']-1,snake[i]['y']);
				}
				snake[i]['x']=snake[i]['x']-1;
				break;
			case 'N':
				if(i==0){
					eat(snake[i]['x'],snake[i]['y']-1);
					checkDie(snake[i]['x'],snake[i]['y']-1);
				}
				snake[i]['y']=snake[i]['y']-1;
				break;
		}
	}

	$("#mapItem_"+snake[0]['x']+'_'+snake[0]['y']).addClass('snakehead');
}

function corners(x,y){
	var _d='';
	for(var i in corner){
		if(corner[i]['x']==x&&corner[i]['y']==y){
			_d=corner[i]['d'];
		}
	}
	return _d;
}

function clearCorners(x,y){
	if(corner[0]['x']==x&&corner[0]['y']==y){
		$("#mapItem_"+corner[0]['x']+'_'+corner[0]['y']).removeClass('corner');
		corner.shift();
		cornerNum=corner.length;
	}
}

function checkDie(x,y){
	if(x<0||x>mapItemX-1||y<0||y>mapItemY-1){
		clearInterval(timer);
		clearInterval(timeiner);
		var _scores=scores();
		alert('你死了！总分：'+_scores.toFixed(2));
		location.reload();
		return false;
	}
	for(var i in snake){
		if(snake[i]['x']==x&&snake[i]['y']==y){
			clearInterval(timer);
			clearInterval(timeiner);
			var _scores=scores();
			alert('你死了！总分：'+_scores.toFixed(2));
			location.reload();
			return false;
		}
	}
}

function eat(x,y){
	if(mouseX==x&&mouseY==y){
		var _snakeLast=snake[snakeLen-1];
		snake[snakeLen]=new Array();
		switch(_snakeLast['d']){
			case 'E':
				snake[snakeLen]['x']=_snakeLast['x']-1;
				snake[snakeLen]['y']=_snakeLast['y'];
				break;
			case 'S':
				snake[snakeLen]['x']=_snakeLast['x'];
				snake[snakeLen]['y']=_snakeLast['y']-1;
				break;
			case 'W':
				snake[snakeLen]['x']=_snakeLast['x']+1;
				snake[snakeLen]['y']=_snakeLast['y'];
				break;
			case 'N':
				snake[snakeLen]['x']=_snakeLast['x'];
				snake[snakeLen]['y']=_snakeLast['y']+1;
				break;
		}
		snake[snakeLen]['d']=_snakeLast['d'];
		snakeLen=snake.length;
		$("#snake_len").html(snakeLen);
		$("#mapItem_"+mouseX+"_"+mouseY).removeClass('mouse');
		initMouse();
		changeSpeed();
	}
}

function changeSpeed(){
	if((snakeLen-5)%20==0){
		speed=speed-10<30?30:speed-10;
		$("#snake_speed").html(speed);
		clearInterval(timer);
		timer=setInterval(move,speed);
	}
}

function initMouse(){
	var _x,_y;
	var _this=this;
	
	this.randoms=function(){
		_x=Math.floor(Math.random()*(mapItemX-1));
		_y=Math.floor(Math.random()*(mapItemY-1));
		_this.checkMouse();
	}
	this.checkMouse=function(){
		var _e=false;
		for(var i in snake){
			if(snake[i]['x']==_x&&snake[i]['y']==_y){
				_e=true;
				break;
			}
		}
		if(_e==true){
			_this.randoms();
		}else{
			mouseX=_x;
			mouseY=_y;
			$("#mapItem_"+mouseX+'_'+mouseY).addClass('mouse');
		}
	}
	this.randoms();
}

function keyDirection(e){
	var evt = e ||window.event; 
	var key=evt.which||evt.keyCode;
	var notdo=false; 
	var _snakeMoveDirection=snakeMoveDirection;
	switch(key){
		case 37:
			if(snakeMoveDirection=='W'){
				notdo=true;
			}else{
				snakeMoveDirection='W';
			}
			break;
		case 38:
			if(snakeMoveDirection=='N'){
				notdo=true;
			}else{
				snakeMoveDirection='N';
			}
			break;
		case 39:
			if(snakeMoveDirection=='E'){
				notdo=true;
			}else{
				snakeMoveDirection='E';
			}
			break;
		case 40:
			if(snakeMoveDirection=='S'){
				notdo=true;
			}else{
				snakeMoveDirection='S';
			}
			break;
	}
	

	this.createCorner=function(){
		corner[cornerNum]=new Array();
		corner[cornerNum]['x']=snake[0]['x'];
		corner[cornerNum]['y']=snake[0]['y'];
		corner[cornerNum]['d']=snakeMoveDirection;
		$("#mapItem_"+corner[cornerNum]['x']+'_'+corner[cornerNum]['y']).addClass('corner');
		cornerNum++;
	}

	if(notdo==false){
		if(cornerNum>0){
			if(corner[cornerNum-1]['x']!=snake[0]['x']||corner[cornerNum-1]['y']!=snake[0]['y']){
				this.createCorner();
			}else{
				snakeMoveDirection=_snakeMoveDirection;
			}
		}else{
			this.createCorner();
		}
	}
}

function timeing(){
	if(timeSecond<60){
		timeSecond++;
		$("#second").html(timeSecond);
	}else{
		timeMinute++;
		$("#minute").html(timeMinute);
		timeSecond=0;
		$("#second").html(timeSecond);
	}
	timestr++;
}

function scores(){
	var _snakeLen=snakeLen-5
	return Math.abs(_snakeLen*_snakeLen-timestr)/timestr*1000/speed;
}
```
