<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<script src="jquery-1.7.1.min.js"></script>
<title>毛毛球</title>
<style>
table{
border:1px solid blue;
position:relative;
}
table td{
border:1px solid blue;
width:25px;
height:25px;
padding:0;
margin:0;
border-collapse:collapse;
vertical-align:middle;
text-align:center;
cursor:pointer;
}
.fling{
width:25px;
height:25px;
background-color:red;
position:absolute;
line-height: 23px;
text-align: center;
}
</style>
</head>

<body>
<div id="test"></div>
<div id='c'></div>
<input type="button" onclick="Earth.clear()" value="clear" />
<input type="button" onclick="Earth.play()" value="play" />
<input type="button" onclick="Earth.show()" value="show" />
<center>
<div id="theEarth"></div>
<p id="console"></p>
<center>
</body>
<script>
var Util={
        isDebug:true,
        debug:function(msg){
                if(this.isDebug && console && console.log)
                console.log(msg);
        }
}
var Earth={
        x:7,//宽
        y:8,//高
        path:[],//成功路径
        flings:[],//地球上的毛毛球
        inAction:false,
        E:'500px',
        S:'500px',
        W:'0',
        N:'0',
        animate:[],
        putFling:function(x,y){
                if(this.findFling(x,y)>=0){
                        this.popFling(x,y);
                }else{
                        var id=Earth.flings.length;
                        var t=$('tr:eq('+(y-1)+') > td:eq('+(x-1)+')');
                        //t.html(id+1);
                        var offset=t.offset();
                        offset.left++;offset.top++;
                        var f=$('<div class="fling">'+(id+1)+'</div>');
                        f.offset(offset);
                        $('body').append(f);
                        Earth.flings.push(new Fling(id,x,y,Earth.flings,f));
                }
        },findFling:function(x,y){
                for(var i=0;i<this.flings.length;i++){
                        if(this.flings[i].x==x &&this.flings[i].y==y){
                                return i;
                        }
                } 
                return -1;
        },popFling:function(x,y){
                var i=this.findFling(x,y);
                if(i>=0){
                        $('tr:eq('+(y-1)+') > td:eq('+(x-1)+')').html('');
                        this.flings.splice(i,1);
                }
        },
        clear:function(){//清除
                /*Earth.flings=[];
                Earth.path=[];
                this.inAction=false;
                $(document).clearQueue('fling');
                $('td',Earth.e).html('');
                $('#console').html('');
                $('.fling').remove();*/
                window.location.reload(true);
        },
        init:function(c,x,y){
                Earth.y=y;
                Earth.x=x;
                var t=$('<table cellspacing="0" cellpadding="0">');
                var sb=[];
                for(var i=0;i<y;i++){
                        sb.push('<tr>');
                        for(var j=0;j<x;j++){
                                sb.push('<td></td>');
                        }
                        sb.push('</tr>');
                }
                t.html(sb.join(''))
                t.appendTo('#'+c);
                $('td',t).click(function(){
                        Earth.putFling(this.cellIndex+1,this.parentNode.rowIndex+1);
                });
                Earth.e=t;
        },
        getTargetFling:function(flings,fling,d){
                 var tf=Earth.getFling(flings,fling.x,fling.y,d);
                 if(tf!=null && fling.pre==null && (Math.abs(fling.x-tf.x)==1 || Math.abs(fling.y-tf.y)==1)){//相邻的球
                         return null;
                 }else{
                         return tf;
                 }
        },
        getFling:function(flings,x,y,d){
                if(d=='E'){
                        x++
                }else if(d=='S'){
                        y++
                }else if(d=='W'){
                        x--
                }else{
                        y--
                }
                if(x>Earth.x || x<0 || y>Earth.y || y<0){
                        return null;
                }
                for(var i=0;i<flings.length;i++){
                        var f=flings[i];
                        if(f.alive && f.x==x&&f.y==y){
                                return f;
                        }
                }
                return Earth.getFling(flings,x,y,d);
        },getCurrentFling:function(flings){
                var c=[];
                for(var i=0;i<flings.length;i++){
                        var f=flings[i];
                        if(f.alive){
                                c.push(new Fling(f.id,f.x,f.y,c,f.substitute));
                        }
                }
                return c;
        },
        play:function(){
                if(Earth.run(Earth.flings)){
                        $('#console').html(Earth.path.join('<br>'));
                }else{
                        $('#console').html('此题无解');
                }
                
        },run:function(flings){
                if(flings.length==1){//只有一个球完成任务
                        return true;;
                }
                for(var i=0;i<flings.length;i++){
                        for(var j=0;j<Direction.length;j++){
                                var _flings=Earth.getCurrentFling(flings);//根据当前场景做下一次尝试
                                var newflings=_flings[i].doMove(Direction[j]);
                                if(newflings && Earth.run(newflings)){//移动后的新场景
                                        return true;
                                }
                                Earth.path.pop();
                        }
                }
                return false
        },show:function(){
                if(this.path.length==0){
                        this.play();
                }
                Earth.inAction=true;
                for(var i=0;i<Earth.path.length;i++){
                        var p=Earth.path[i];
                        Earth.flings[p.id].doMove(p.d);
                }
                this.queue();
                this.dequeue();
        },dequeue:function(){
                $(document).dequeue('fling');
        },queue:function(){
                $(document).queue('fling',Earth.animate);
        }
};
var Direction=['N','E','S','W'];//能移动的4个方向
var Fling=function(id,x,y,flings,substitute){
        this.id=id;
        this.x=x;
        this.y=y;
        this.flings=flings;
        this.substitute=substitute;
        this.alive=true;//false表示不在地球上
        this.pre;//与之相撞的球
        this.path=[];
        this.goto=function(x,y){
                Util.debug('fling '+(this.id+1)+' goto:['+this.x+','+this.y+']->['+x+','+y+']');
                this.x=x;
                this.y=y;
                this.flings.isChanged=true;
                if(Earth.inAction){
                        var t=$('tr:eq('+(this.y-1)+') > td:eq('+(this.x-1)+')');
                        var offset=t.offset();
                        offset.left++;
                        offset.top++;
                        var id=this.id;
                        Earth.animate.push(function(){
                                Earth.flings[id].substitute.animate({left:offset.left,top:offset.top},{queue:false,duration:500,complete:function(){
                                        Earth.dequeue()}});
                        });
                }
        };
        this.doMove=function(d){//执行移动操作
                if(this.pre==null && !Earth.inAction){
                        Earth.path.push({id:this.id,d:d,toString:function(){return (this.id+1)+'->'+this.d}});
                }
                var tf=Earth.getTargetFling(this.flings,this,d);//获取要碰撞的fling
                if(tf){
                        if(d=='E'){
                                this.goto(tf.x-1,this.y);
                        }else if(d=='S'){
                                this.goto(this.x,tf.y-1);
                        }else if(d=='W'){
                                this.goto(tf.x+1,this.y);
                        }else{
                                this.goto(this.x,tf.y+1);
                        }
                        tf.pre=this;
                        tf.doMove(d);
                }else{
                        if(this.pre!=null){
                                this.alive=false;//被撞飞
                                if(Earth.inAction){
                                        var id=this.id,o={opacity:0};
                                        if(d=='E'){
                                                o.left=Earth.E;
                                        }else if(d=='S'){
                                                o.top=Earth.S;
                                        }else if(d=='W'){
                                                o.left=Earth.W;
                                        }else{
                                                o.top=Earth.N;
                                        }
                                        Earth.animate.push(function(){
                                                Util.debug((id+1)+' is out');
                                                Earth.flings[id].substitute.animate(o,{queue:false,duration:300,complete:function(){Earth.dequeue()}});
                                        });
                                }
                        }
                }
                if(this.flings.isChanged){
                        return Earth.getCurrentFling(this.flings);
                }else{
                        return null;
                }
            
        }
}
Earth.init('c',7,8);
</script>
</html>
