<template>
	<button @click="handleClick" :style="styles">
		<slot></slot>
	</button>
</template>
<script type="text/javascript">
	export default{
	props:{
	   color:{
	      type:String,
	      default:'#0c6'
		}
	},
	computed:{
		styles(){
			return {
			background:this.color
		}
		}
	},
	methods:{
	  handleClick(e){
	    this.$emit('click',e)
	}
}
}
	
</script>
<style type="text/css" scoped="scoped">
    button{
      border:0;
      outline:none;
      color:#fff;
      padding:4px 8px;
}
button:active{
	position:relative;
	top:1px;
	left:1px;
}
</style>