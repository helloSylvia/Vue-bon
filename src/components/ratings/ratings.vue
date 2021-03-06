<template>
	<div class="ratings" ref="ratings">
		<div class="rating-content" >
			<div class="overview">
				<div class="code">
					<h1 class="scroe">{{ seller.score }}</h1>
					<div class="title">综合评分</div>
					<div class="rank">高于周边商家{{ seller.rankRate }}%</div>
				</div>
				<div class="rating">
					<div class="score-wrapper">
						<span class="title">服务态度</span>
						<star :size="36" :score="seller.serviceScore"></star>
						<span class="score">{{ seller.serviceScore }}</span>
					</div>
					<div class="score-wrapper">
						<span class="title">商品评分</span>
						<star :size="36" :score="seller.foodScore"></star>
						<span class="score">{{ seller.foodScore }}</span>
					</div>
					<div class="delivery-wrapper">
						<span class="title">送达时间</span>
						<span class="delivery">{{ seller.deliveryTime }}分钟</span>
					</div>
				</div>
			</div>
			<split></split>
			<ratingselect :selectType="selectType" :onlyContent="onlyContent" 
					 :ratings="ratings" @select="selectRating" @toggle="toggelContent"></ratingselect>
			<div class="rating-wrapper">
				<ul>
					<li v-for="rating in ratings" class="rating-item" v-show="needShow(rating.rateType,rating.text)">
						<div class="picture">
							<img :src="rating.avatar" width="28" height="28">
						</div>
						<div class="content">
							<h1 class="name">{{ rating.username }}</h1>
							<div class="star-wrapper">
								<star :size="24" :score="rating.score"></star>
								<span class="delivery" v-show="rating.deliveryTime">{{ rating.deliveryTime}}</span>
							</div>
							<p class="text">{{ rating.text }}</p>
							<div class="recommend" v-show="rating.recommend && rating.recommend.length">
								<span class="icon-thumb_up"></span>
								<span class="item" v-for="item in rating.recommend">{{ item }}</span>
							</div>
							<div class="time">
								{{ rating.rateTime | formatDate }}
							</div>
						</div>
					</li>
				</ul>
			</div>
		</div>
		
	</div>
</template>

<script>
	import BScroll from 'better-scroll';
	import star from 'components/star/star'; 
	import {formatDate} from 'common/js/date.js';
	import split from 'components/split/split';
	import ratingselect from 'components/ratingselect/ratingselect';

	const ALL = 2;
	const ERR_OK = 0;

export default {
	props: {
		seller: {
			type: Object
		}
	},
	components: {
		star,
		split,
		ratingselect
	},
	data() {
		return {
			// 需要ratings 从后台传过来的
			ratings:[],
			selectType: ALL,
			onlyContent: true,
		}
	},
	created() {
		this.$http.get('/api/ratings').then((response) => {
			response = response.body;
			if(response.errno === ERR_OK){
				this.ratings = response.data
				this.$nextTick(() => {
					this.scroll	= new BScroll(this.$refs.ratings,{
						click: true
					})
				})
			}
		})
	},
	filters: {
			formatDate(time) {
				let date = new Date(time);
				return formatDate(date,'yyyy-MM-dd hh:mm');
			}
	},
	methods: {
		needShow(type,text) {
			// 只显示有内容的
			if(this.onlyContent && !text){
				return false;
			}
			// 选择的类型
			if(this.selectType === ALL){
				return true;
			}else{
				return type === this.selectType;
			}
		},
		// 参数不用写在上面的元素绑定中，其中自带
		selectRating(type) {
			this.selectType = type
			// 内容会变化，高度会变化，dom在更新，下一个时间周期才会改变
			this.$nextTick(() => {
				this.scroll.refresh();
			})
		},
		toggelContent() {
			this.onlyContent = !this.onlyContent
			this.$nextTick(() => {
				this.scroll.refresh();
			})
		}
	}
}

</script>

<style lang="stylus" rel="stylesheet/stylus" >
@import "../../common/stylus/mixin.styl"
	.ratings
		position: absolute
		top: 174px
		bottom: 0
		left: 0
		width: 100%
		overflow: hidden
		.overview
			display: flex
			padding: 18px 0
			.code
				flex: 0 0 137px
				padding: 6px 0
				// 兼容
				width: 137px
				border-right: 1px solid rgba(7,17,27,0.1)
				text-align: center
				@media only screen and (max-width:320px)
					flex: 0 0 120px
					width: 120px
				.scroe
					margin-bottom: 6px
					line-height: 28px
					font-size: 24px
					color: rgb(255,153,0)
				.title
					margin-bottom: 8px
					line-height: 12px
					font-size: 12px
					color: rgb(7,17,27)
				.rank
					line-height: 10px
					font-size: 10px
					color: rgb(147,153,159)
					
			.rating
				flex: 1
				padding: 6px 0 6px 24px
				@media only screen and (max-width:320px)
					padding-left: 6px
				.score-wrapper
					margin-bottom: 8px
					font-size: 0
					.title
						display: inline-block
						vertical-align: top
						line-height: 18px
						font-size: 12px
						color:rgb(7,17,27)
					.star
						display: inline-block
						margin: 0 12px
						vertical-align: top
					.score
						line-height: 18px
						display: inline-block
						vertical-align: top
						font-size: 12px
						color: rgb(255,153,0)
				.delivery-wrapper
					font-size: 0
					.title
						line-height: 18px
						font-size: 12px
						color:rgb(7,17,27)
					.delivery
						margin-left: 12px
						font-size: 12px
						color: rgb(147,153,159)
		.rating-wrapper
			padding: 0 18px
			.rating-item
				display: flex
				padding: 18px 0
				border-1px(rgba(7,17,27,0.1))
				.picture
					flex: 0 0 28px
					width: 28px
					margin-right: 12px
					img
						border-radius: 50%
				.content
					position: relative
					flex: 1
					.name
						margin-bottom: 4px
						line-height: 12px
						font-size: 10px
						color:rgb(7,17,27)
					.star-wrapper
						margin-bottom: 6px
						font-size: 0
						.star
							margin-right: 6px
							display: inline-block
							vertical-align: top
						.delivery
							display: inline-block
							vertical-align: top
							line-height: 12px
							font-size: 10px
							color:rgb(147,152,159)
					.text
						margin-bottom: 8px
						line-height: 18px
						color: rgb(7,17,27)
						font-size: 12px
					.recommend
						line-height: 16px
						font-size: 0
						.icon-thumb_up, .item
							display: inline-block
							margin: 0 8px 4px 0
							font-size: 9px
						.icon-thumb_up
							color: rgb(135,142,205)
						.item
							padding: 6px
							border: 1px solid rgba(7,17,27,0.1)
							border-radius: 1px
							color: rgb(147,153,159)
					.time
						position: absolute
						top: 0
						right: 0
						line-height: 12px
						font-size: 10px
						color: rgb(147,153,159)
							
						
						
										
				
				
				
	
</style>








