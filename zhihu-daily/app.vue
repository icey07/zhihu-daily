<template>
	<div class="daily">
		<div class="daily-menu">
			<div class="daily-menu-item" :class="{on:type==='recommend'}" @click="handleToRecommend">每日推荐</div>
			<div class="daily-menu-item" :class="{on:type==='daily'}" @click="showThemes=!showThemes">主题日报</div>
			<ul v-show="showThemes">
				<li v-for="item in themes" >
					<a :class="{on:item.id===themeId&&type==='daily'}" @click="handleToTheme(item.id)">{{item.name}}</a>
				</li>
			</ul>
			
		</div>
		<div class="daily-list" ref="list">
			<template v-if="type==='recommend'">
				<div v-for="list in recommendList">
					  <div class="daily-date">{{formatDay(list.date)}}</div>
					  <Item v-for="item in list.stories" :data="item" :key="item.id" @click.native="handleClick(item.id)"></Item>
				</div>
			</template>
			 <template v-if="type==='daily'">
				<Item v-for="item in list" :data="item" :key="item.id" @click.native="handleClick(item.id)"></Item>
			</template> 
		</div>	
		<daily-article :id="articleId"></daily-article>
	</div>
</template>
<script type="text/javascript">	
	import $ from './libs/util';
	import Item from './components/item.vue';
	import dailyArticle from './components/daily-article.vue';

	export default{
	   components:{Item,dailyArticle},
	   data(){
	     return {
	        type:'recommend',
	        showThemes:false,
	        themes:[],
	        themeId:0,
	        recommendList:[],
	        dailyTime:$.getTodayTime(),
	        isLoading:false,
	        list:[],
	        articleId:0
	 }
	},
	methods:{
        getThemes(){
          $.ajax.get('themes').then(res=>{
              this.themes=res.others;
              
      		})
    	},
    	handleToTheme(id){
           this.type='daily';
           this.themeId=id;
           this.list=[];
           $.ajax.get('theme/'+id).then(res=>{
              this.list=res.stories.filter(item=>item.type!==1);
              //console.log(this.list)
          	})
    	},
    	handleToRecommend(){
         this.type='recommend';
         this.recommendList=[];
         this.dailyTime=$.getTodayTime();
         this.getRecommendList();
  	  },
  	  getRecommendList(){
         this.isLoading=true;
         const prevDay=$.prevDay(this.dailyTime+24*60*60*1000);
         console.log(prevDay);
         $.ajax.get('news/before/'+prevDay).then(res=>{
           this.recommendList.push(res);
           this.isLoading=false;
       })
  		},
  		formatDay(date){
         let month=date.substr(4,2);
         let day=date.substr(6,2);
         if(month.substr(0,1)==='0'){
             month=month.substr(1,1);
         }
         if(day.substr(0,1)==='0'){
             day=day.substr(1,1)
         }
          return `${month}月${day}日`
  	  },
  	  handleClick(id){
  	    this.articleId=id;
  	}
	},
	mounted(){
	   this.getThemes();
	   this.getRecommendList();
     
     //获取Dom
	   const $list=this.$refs.list;

	   //监听中栏中的滚动时间
	   $list.addEventListener('scroll',()=>{
	       //在主题日报或正在加载推荐列表时停止操作
	       if(this.type==='daily'||this.isLoading)
	           return;
	        //已经滚动的距离加上页面的高度等于整个内容区域高总时，视为接触底部
         if($list.scrollTop+document.body.clientHeight>=$list.scrollHeight){
            this.dailyTime-=24*60*60*1000;
            this.getRecommendList();
       }
	 });
	}
}
</script>