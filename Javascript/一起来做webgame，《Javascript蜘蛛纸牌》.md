# 一起来做webgame，《Javascript蜘蛛纸牌》

> 写于：2013-09-22

不得不说，做游戏是会上瘾的，这次带来的是win系统上的经典游戏《蜘蛛纸牌》，不能完美，但求一玩

## 关于蜘蛛纸牌

规则请打开win系统的蜘蛛纸牌，然后点击帮助

## 这里要实现的

- 同样是两副牌，一共104张
- 同一种花色的低难度游戏

## 需要解决的问题

1. 洗牌
2. 判断点击牌所在序列是否符合可移动条件
3. 判断目标位置是否符合可移动条件
4. 移动符合条件的纸牌序列到目标位置
5. 完成一个完整序列时的清除
6. 发牌

综合起来，《蜘蛛纸牌》基本上就这么6个问题，解决了，也就完成了。下面一个一个来实现

## 1、洗牌

光洗牌，不难。如果要做到每次洗牌都有解就不是我能解决的问题了（win系统里的蜘蛛纸牌是不是每次都有解，我确实不知道）。这里就随便洗洗，没解也没办法，即使有解，你也不一定能解完，是吧。

这里是同色，所以不用去考虑4个花色，那一副牌，从A-K，是13张，有4组，就是13*4=52张牌。两副就是104张。先初始化一组牌

```
var _cards_init=['1','2','3','4','5','6','7','8','9','10','J','Q','K'];
```

跟着把它变成1副牌

```
_cards_init=_cards_init.concat(_cards_init,_cards_init,_cards_init);  //使用concat来链接数组
```

这里自己链接自己3次，就变成了4组，一副牌。再接着链接自己一次，就变成两副牌

```
_cards_init=_cards_init.concat(_cards_init);
```

两副牌有了，接着就洗牌，这里简单的使用随机数来洗。

```
//洗牌
function shuffle(){
	var len=_cards_init.length;
	if(len>0){
		var i=random(len);
		_cards.push(_cards_init[i]);
		_cards_init.splice(i,1);
		shuffle();
	}
}
```

这是一个递归，每次从初始化的剩余牌序列里随机取一个，放到新牌的数组里，然后将取走的牌从初始化的数组里移除，重计长度，只要长度不小于1，那么重复这个操作，至到全部取完为止。这样生成的新牌数组里的牌基本上就是乱序了，达到洗牌的效果。

牌洗好之后，就得将最开始的牌面放到界面上。win7的蜘蛛纸牌是10列，前4列5张扣牌，1张明牌，后6列4张扣牌，1张明牌。这里也按这样的方式开始。

因为是按列为基础进行摆牌，所以程序在确定牌所在位置的时候，需要分别记录，每一列等于是一个序列，有牌移进来，要增加它的长度，有牌移走的时候，要减少它的长度。所以这里使用了一个数组来分别记录牌的信息

```
var _cards_table=new Array();
```

接着确定位置，按照初始化的界面，一共要放54张牌到牌面上，其中44张是扣牌，10张是明牌，所以，先从新牌数组里取54张牌出来，其中前10张，也就是0-9，是个位数，直接将下标做了牌的参数

```
for(var i=0;i<10;i++){
	_cards_table[i]=new Array();
}
var _html_cards_table='';

for(var i=0;i<54;i++){
	if(i<=9){
		j=i;
		_html_cards_table+="<div class='cardbg cardbg"+j+"' action-t='' action-line='"+j+"'></div>";
	}else{
		var s=i.toString();
		j=s.substring(s.length-1);
	}
	j=parseInt(j);
	var num=cardNum(_cards[i]);
	var back='';
	//背面牌
	if(i>=0&&i<44){
		back='back';
	}
	var l=_cards_table[j].length;
	_html_cards_table+="<div class='card cardv"+num+" card"+j+" "+back+"' action-t='"+back+"' action-line='"+j+"' action-index='"+l+"' style='z-index:1;position:absolute;left:"+_cardsPostion[0]+"px;top:"+_cardsPostion[1]+"px;'>"+_cards[i]+"</div>";
	_cards_table[j].push(num);
}
```

其实，这里固定牌的位置，直接使用了CSS样式，不太好，因为有时候你不知道到底一列会出现多少个纸牌，要是CSS样式写少了，那多出来的纸牌放的位置就会出现问题。最好的办法是JS直接判断索引值，根据索引值*固定数来确定位置。

这些牌放出去后，自然得将它们从新牌的数组里移除，以后发牌，就是数组里剩下的牌了

```
_cards.splice(0,54);
_cards_table+='<div id="post">发牌<span id="postnum">5</span></div>';  //顺便添加一个发牌的按钮
$("#game_map").html(_cards_table);  //将生成的牌放到游戏地图上
```
这里除了这些初始牌面外，还有一个东西，那就是隐藏的一个牌，因为每列的牌都有可能完全的移出，这时候符合条件的牌序列又可以移到这个空列里来，所以，得在那个位置放一个看不见的牌，但这个牌只有列号，没有其它的东西。

```
if(i<=9){
	j=i;
	_cards_table+="<div class='cardbg cardbg"+j+"' action-t='' action-line='"+j+"'></div>";
}
```

## 2、判断点击牌所在序列是否符合可移动条件。3、判断目标位置是否符合可移动条件

首先，得为各牌添加单击事件。然后第一次点的牌要记录下它的3个东西：1、所在列。2、列上的索引值。3、牌面值（A-K）。再次点击其它牌的时候，进行判断，它是不是符合移动条件，如：被点击牌及之后的牌序列是不是符合条件（递减），目标牌是不是比第一次点击牌大1，如果是，移动牌到新位置，如果不是，给出提示，不符合条件

```
function clickCard(){
	$(".card").unbind('click').click(function(){
		var line=$(this).attr('action-line');
		var back=$(this).attr('action-t');
		if(back){
			return false;
		}
		var index=parseInt($(this).attr('action-index'));
		var cards_num;
		line=parseInt(line);
		cards_num=_cards_table[line][index];
		if(_cards_selected.length==1){
			if((cards_num-1)==_cards_selected[0][2]){
				var _index=index+1;
				moveCard(line,_index,_cards_selected[0][2]);
			}else{
				alert('不能移动到那个位置2_'+cards_num+'_'+_cards_selected[0][2]);
			}
			css_remove();
			_cards_selected.shift();
			clickCard();
		}else{
			if(getCards(line,index)==0){
				_cards_selected[0]=[];
				_cards_selected[0][0]=line;
				_cards_selected[0][1]=index;
				_cards_selected[0][2]=cards_num;
			}else{
				css_remove();
			}
		}
	});
}
```

## 4、移动符合条件的纸牌序列到目标位置

移走了，也就得把它从该列的数组里清除掉，这里使用splice()，只需要传入起始索引值，将它和它之后的所有牌都清除。然后创建新的牌加入到目标列的数组里

```
function moveCard(line,index,cards_num){
	line=parseInt(line);
	var len=_cards_table[_cards_selected[0][0]].length;
	_cards_table[_cards_selected[0][0]].splice(_cards_selected[0][1],len);
	var html='';
	var moveCardIndex=index;
	_moveCardNum=len-_cards_selected[0][1];
	for(var i=_cards_selected[0][1];i<len;i++){
		var v=$(".card"+_cards_selected[0][0]).eq(_cards_selected[0][1]).html();
		var o=$(".card"+_cards_selected[0][0]).eq(_cards_selected[0][1]);
		var num=cardNum(o.html());
		_cards_table[line].push(num);
		var cardline=o.attr('action-line');
		o.removeClass('card'+cardline);
		o.addClass('card'+line);
		o.attr('action-line',line);
		o.attr('action-index',moveCardIndex);
		move_animation(o,line,moveCardIndex,1);
		moveCardIndex++;
	}
	$(".card"+_cards_selected[0][0]).eq(_cards_selected[0][1]-1).removeClass('back');
	$(".card"+_cards_selected[0][0]).eq(_cards_selected[0][1]-1).attr('action-t','');
	$("#scores").html(parseInt($("#scores").html())+1);
	
}
```

checkComplate(line);用于判断是否已经实现A-K，如果是将它清除

## 5、完成一个完整序列时的清除

首先判断该列的纸牌数是否超过13张，因为如果连13张都没有，又怎么会有完整的A-K呢。之后再判断明牌是不是有13张，而且明显的大小是不是从大到小减1的。如果都符合，那将它们全部清除，并从该列的数组里将它们清除。

```
function checkComplate(line){
	var obj=$(".card"+line)
	var len=obj.length;
	var preNum=0;
	var error=0;
	var comNum=[];
	if(len>=13){
		for(var i=0;i<len;i++){
			if(obj.eq(i).attr('action-t')==''){
				if(preNum>0){
					if(parseInt(cardNum(obj.eq(i).html()))+1==preNum){
					}else{
						comNum=[];
					}
					comNum.push(i);
				}else{
					comNum.push(i);
				}
				preNum=parseInt(cardNum(obj.eq(i).html()));
			}
		}
		if(comNum.length>=13){
			for(var i in comNum){
				obj.eq(comNum[i]).remove();
				if(i==0){
					obj.eq(comNum[i]-1).removeClass('back');
					obj.eq(comNum[i]-1).attr('action-t','');
					var len=_cards_table[line].length;
					_cards_table[line].splice(comNum[i],len);
				}
			}
			_left_group--;
			if(_left_group==0){
				alert('你赢了，共移动'+$('#scores').html()+'次');
				$("#re").click();
			}
		}
	}
}
```

## 6、发牌

发牌其实和初始化牌面没什么区别，只是每次只发10张，也就是向0-9列分别增加一张牌，发出去后，剩于牌就减去这些牌

```
function post(){
	$("#post").click(function(){
		if(_cards.length<=0){
			alert('无牌可发了');
			return false;
		}
		var _html_cards_table='';
		for(var i=0;i<10;i++){;
			var num=cardNum(_cards[i]);
			var l=_cards_table[i].length;
			_html_cards_table+="<div class='card cardv"+num+" card"+i+"' action-t='' action-index='"+l+"' action-line='"+i+"' style='z-index:1;position:absolute;left:"+_cardsPostion[0]+"px;top:"+_cardsPostion[1]+"px;'>"+_cards[i]+"</div>";
			_cards_table[i].push(num);
		}
		_cards.splice(0,10);
		var cardbenum=$(".card").length;
		$("#game_map").append(_html_cards_table);
		var _i=cardbenum;
		var _this=this;
		this.move_animation=function(){
			var line=$(".card").eq(_i).attr('action-line');
			var index=$(".card").eq(_i).attr('action-index');
			move_animation($(".card").eq(_i),line,index);
			_i++;
			if(_i<cardbenum+10){
				setTimeout(_this.move_animation,100);
			}
		}
		this.move_animation();
		$("#postnum").html(parseInt($("#postnum").html())-1);
		clickCard();
	});
}
```

以上6点，就是《蜘蛛纸牌》的基本内容，代码里还有一些方法，是为了增加一些方便，比如返回J-K的值等等，这里就不说了。直接编辑源代码，还是很累的。

里面还有关于动画的内容，暂时就不讲了，全部代码也不帖了，可以查看试玩页面的源文件
