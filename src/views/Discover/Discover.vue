<template>
    <div class="discover" v-loading="loading">  
        <div class="Carousel">
            <el-carousel :interval="3000" type="card" height="250px">
                <el-carousel-item v-for="(item,index) in banners" :key="index">
                    <img class="item-img" :src="item.imageUrl" alt="banner" width="100%" height="100%" >
                </el-carousel-item>
            </el-carousel>            
        </div>
        <div class="songs-wrap">
            <h4>推荐歌单</h4>
            <div class="list">
                <ul>
                    <li class="iconfont icon-play" v-for="(item,index) in recommendList" :key="index" @click="toPlaylistDetail(item.id)">
                        <p class="first-p">{{item.copywriter}}</p>
                        <img v-lazy="item.picUrl" alt="recommend">
                        <p class="last-p" :title="item.name">{{item.name}}</p>
                    </li>                                                                                                                    
                </ul>
            </div>
        </div>
        <div class="songs-wrap">
            <h4>最新音乐</h4>
            <ul class="new-songs">
                <li v-for="(item,index) in newSongs" :key="index">
                    <div class="music-img-wrap">
                        <img v-lazy="item.picUrl" alt="newSongs">
                        <p class="iconfont icon-play" @click="playMusic(item)"></p>
                    </div>

                    <div class="music-info">
                        <p class="music-name">{{item.name}}</p>
                        <p class="music-singer" v-for="(singer,i) in item.song.artists" :key="i" style="display:inline;margin:0 5px">{{singer.name}}</p>
                    </div>
                </li>                                                                                                                       
            </ul>
        </div>
        <div class="songs-wrap">
            <h4>推荐MV</h4>
            <ul class="mv-list">
                <li v-for="(item,index) in newMvs" :key="index" @click="toMvDetail(item.id)">
                    <div class="mv-img-wrap">
                        <img v-lazy="item.picUrl" alt="newMvs">
                        <p class="iconfont icon-play play"></p>
                        <p class="play-count iconfont icon-play">{{item.playCount}}</p>
                    </div>
                    <div class="mv-info">
                        <p class="title">{{item.name}}</p>
                        <p class="author">{{item.artistName}}</p>
                    </div>
                </li>                                              
            </ul>
        </div>
    </div>
</template>

<script>
import { bannerAPI,recommendSonglistAPI,recommendSongAPI,recommendMVAPI,playMusicAPI } from '@/utils/api'

export default {
    data(){
        return {
            banners:[],
            recommendList:[],
            newSongs:[],
            newMvs:[],
            loading:true
        }
    },
    computed:{
        musicQueue(){
            return this.$store.state.musicQueue
        }
    },
    created(){
        // 获取轮播图
        bannerAPI().then(res=>{
            this.banners = res.data.banners
        }).then(()=>{
            this.loading = false
        })

        // 获取推荐歌单
        recommendSonglistAPI({limit:10}).then(res=>{
            // console.log(res)
            this.recommendList = res.data.result
        })

        // 获取最新音乐
        recommendSongAPI().then(res=>{
            // console.log(res)
            this.newSongs = res.data.result
        })

        //获取推荐MV
        recommendMVAPI().then(res=>{
            // console.log(res)
            this.newMvs = res.data.result
            for(let i=0;i<this.newMvs.length;i++){
                if(this.newMvs[i].playCount >= 100000)
                    this.newMvs[i].playCount = parseInt(this.newMvs[i].playCount/10000)+'万'
            }             
        })
    },
    methods:{
        timeFormat(time){
            let min = parseInt(time / 60000).toString().padStart(2,'0')
            let second = parseInt((time-min*60000)/1000).toString().padStart(2,'0')
            return min+":"+second
        },
        playMusic(item){
            console.log(item)
            playMusicAPI({id:item.id}).then(res=>{
                // console.log(res)
                if(res.data.data[0].url){
                this.songUrl = res.data.data[0].url

                let musicInfo = {
                    imgUrl:item.picUrl,
                    // singer:item.song.artists[0].name,
                    artistInfo:item.song.artists,
                    songName:item.name,
                    id:item.id,
                    duration:this.timeFormat(item.song.duration)                    
                }
                this.$store.commit("changeMusicUrl",this.songUrl)
                this.$store.commit("changeMusicInfo",musicInfo)
                this.$store.commit("changeMusicStatus",false)                
                this.$store.commit("changeMusicQueue",musicInfo)
                let ids = []
                for (const item of this.musicQueue) {
                    ids.push(item.id)
                }
                this.$store.commit("changeNowIndex",ids.indexOf(musicInfo.id))                 

                // this.$parent.$parent.musicUrl = this.songUrl
                // // 传入歌曲信息
                // this.$parent.$parent.musicInfo = {
                //     imgUrl:item.picUrl,
                //     singer:item.song.artists[0].name,
                //     songName:item.name
                // }

                }else{
                    this.$message({
                    showClose: true,
                    message: '对不起，歌曲暂时无法播放。',
                    type: 'error'
                    });
                }
            })
        },
        toPlaylistDetail(id){
            this.$router.push(`/playlist?id=${id}`)
        },
        toMvDetail(id){
            this.$router.push(`/mvdetail?id=${id}`)
        }        
    }
}
</script>

<style scoped>

    ul {
        list-style: none;
    }

  .Carousel >>> .el-carousel__item:nth-child(2n) {
    background-color: #99a9bf;
  }
  
  .Carousel >>> .el-carousel__item:nth-child(2n+1) {
    background-color: #d3dce6;
  }

  .Carousel >>> .el-carousel__button {
      width: 8px;
      height: 8px;
      border-radius: 50%;
  }

  .Carousel >>> .el-carousel__indicator.is-active button {
      background-color: rgb(236, 65, 65);
  }

  .discover {
      max-width: 1300px;    
      margin: 0 auto;
      padding: 20px;
  }

  .discover >>> .el-loading-spinner {
      top: 20%;
  }
  
  .el-carousel__item--card{
      border-radius: 20px;
  }

  .songs-wrap {
      margin-bottom: 20px;
  }

  .songs-wrap h4 {
      color: #333;
      margin-bottom: 20px;
  }

  .songs-wrap .list ul{
      width: 100%;
      display: flex;
      justify-content: space-between;
      flex-wrap: wrap;
  }

  .songs-wrap .list li {
      width: 18%;
      margin: 10px 0;
      position: relative;
      overflow-y: hidden;
  }

  .list li .first-p {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      background-color: rgba(0, 0, 0, .5);
      color: #fff;
      font-size: 12px;
      padding: 5px;
      box-sizing: border-box;
      /* border-top-left-radius: 10px;
      border-top-right-radius: 10px; */
      transform: translateY(-100%);;
      transition: .5s;
  }

  .list li::before {
      content: "\e665"; 
      position: absolute;
      bottom: 25px;
      right: 5px;
      width: 35px;
      height: 35px;
      background-color: rgba(255, 255, 255, .8);
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      /* font-size: 35px; */
      color: #c0392b;
      opacity: 0;
      transition: .3s;
      cursor: pointer;
  }

  .list li:hover .first-p{
      transform: translateY(0);
  }

    .list li:hover::before{
        opacity: 1;
  }

  .songs-wrap ul img {
      width: 100%;
      border-radius: 5px;
      /* opacity: 1; */
  }

  .songs-wrap ul .last-p {
      font-size: 14px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
  }

    /* 最新音乐  */
  .new-songs {
      width: 100%;
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
  }

  .new-songs li {
      width: 50%;
      display: flex;
      padding: 10px;
      box-sizing: border-box;
      position: relative;
  }

  .new-songs li:hover {
      background-color: #ddd;
  }

   .new-songs li:hover::before {
       opacity: 1;
  }

    .music-img-wrap {
        position: relative;
        width: 100px;
        cursor: pointer;
    }

    .music-img-wrap:hover p::before {
        opacity: 1;
    }

  .music-img-wrap p::before {
      content: "\e665"; 
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%,-50%);
      width: 35px;
      height: 35px;
      background-color: rgba(255, 255, 255, .8);
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #c0392b;
      opacity: 0;
      transition: .3s;
      cursor: pointer;
  }

  .music-info {
      flex: 1;
      padding: 0 10px;
  }

  .new-songs li p:first-child {
      margin-bottom: 20px;
  }

  .music-singer {
      color: #a5a1a1;
  }


    /* 推荐MV  */
  .mv-list {
      display: flex;
      justify-content: space-between;
  }

  .mv-list li {
      width: 25%;
      padding: 0 10px;
  }

  .mv-img-wrap {
      position: relative;
      cursor: pointer;
  }

  .mv-img-wrap:hover .play::before {
      opacity: 1;
  }

  .mv-img-wrap .play::before {
      content: "\e665"; 
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%,-50%);
      width: 35px;
      height: 35px;
      background-color: rgba(255, 255, 255, .8);
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #c0392b;
      opacity: 0;
      transition: .3s;
      cursor: pointer;
  }

  .play-count{
    position: absolute;
    top: 5px;
    right: 5px;
    color: #fff;
    text-shadow: 0 0 2px rgb(0, 0, 0);      
  }

  .play-count::before {
      margin-right: 5px;
  }

 .mv-info p{
     margin: 5px 0;
 }

  .mv-info .author{
      color: #a5a1a1;
      font-size: 12px;
  }
</style>