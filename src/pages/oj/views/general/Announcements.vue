<template>
  <Row type="flex" :gutter="18">
    <Col :span="containerSpan">
    <!-- <Col :xs="{order:2, span:24}" :sm="{order:2, span:24}" :md="{order:1, span:containerSpan}" style="padding-bottom:20px;"> -->
      <Panel shadow :padding="10" >
        <div slot="title">
          {{title}}
        </div>
        <div slot="extra">
          <Button v-if="listVisible" type="info" @click="init" :loading="btnLoading"><span class="ivu-icon ivu-icon-refresh"></span> {{$t('m.Refresh')}}</Button>
          <Button v-else icon="ios-undo" @click="goBack">{{$t('m.Back')}}</Button>
        </div>
        <transition-group name="announcement-animate" mode="in-out">
          <div class="no-announcement" v-if="!announcements.length" key="no-announcement">
            <p>{{$t('m.No_Announcements')}}</p>
          </div>
          <template v-if="listVisible">
            <ul class="announcements-container" key="list">
              <li v-for="announcement in announcements" :key="announcement.title">
                <!-- <div class="flex-container">
                  <div class="title"><a class="entry" @click="goAnnouncement(announcement)">
                    {{announcement.title}}</a></div>
                  <div class="date">{{announcement.create_time | localtime }}</div>
                  <div class="creator"> {{$t('m.By')}} {{announcement.created_by.username}}</div>
                </div> -->
                  <Row>
                    <Col flex="8" class="flex-container">
                    <div class="title"><a class="entry" @click="goAnnouncement(announcement)">
                      {{announcement.title}}</a></div>
                    </Col>
                    <Col flex="2">
                    <div class="date">{{announcement.create_time | localtime }}</div>
                    </Col>
                    <Col flex="2">
                    <div class="creator"> {{$t('m.By')}} {{announcement.created_by.username}}</div>
                    </Col>
                  </Row>
              </li>
            </ul>
            <Pagination v-if="!isContest"
                        key="page"
                        :total="total"
                        :page-size="limit"
                        @on-change="getAnnouncementList">
            </Pagination>
          </template>

          <template v-else>
          <div v-katex v-html="announcement.content" key="content" class="content-container markdown-body"></div>
          </template>
        </transition-group>
      </Panel>
      <Row v-if="!isContest" type="flex" :gutter="10" style="margin-top: 70px;">
            <!-- <Col  :span="12"> -->
            <Col flex="1 1 400px" style="padding-bottom:20px;">
              <Panel shadow style="padding-top: 10px; padding-bottom: 10px; min-height: 100px;">
                <div slot="title">Bài tập mới</div>
                <ul style="margin-left: 40px;margin-bottom: 20px;">
                  <li style="padding: 5px 0px;"  v-for="p in problemList" :key="p.id">
                    <a class="link-style" :href="'/problem/' + p._id">{{p._id}} - {{p.title}}</a>
                  </li>
                </ul>
              </Panel>
            </Col>
            <!-- <Col  :span="12"> -->
            <Col flex="1 1 400px">
              <Panel shadow style="padding: 10px; min-height: 100px;">
                <div slot="title">{{$t('m.TagsTitle')}}</div>
                <Button v-for="tag in tagList"
                        :key="tag.name"
                        :disabled="query.tag === tag.name"
                        shape="circle"
                        class="tag-btn"><a class="link-style" :href="'/problem?tag=' + tag.name">{{tag.name}}</a>
                </Button>
              </Panel>
            </Col>
        </Row>
    </Col>
    <Col :span="6" v-if="!isContest" >
    <!-- <Col :xs="{order:1, span:24}" :sm="{order:1, span:24}" :md="{order:2, span:6}" v-if="!isContest" style="padding-bottom:20px;"> -->
      <Col>
      <Panel shadow>
        <div style="font-size:14px; text-align:center; width:100%; line-height:16px; background: transparent; color:#636e72;">Không ngừng luyện tập!</div>
        <div class="today">
          <div class="nowWeek">{{nowWeek}}</div>
          <div class="nowDate">
            {{nowDate}}
          </div>
        </div>
        <div v-if="days" style="margin:0 auto; margin-bottom:15px; font-size:12px; text-align:center; width:160px; line-height:16px; background: transparent; color:#636e72;">Bạn đã có <strong>{{days}} </strong> chuỗi ngày học</div>
        <div style="margin-top:-10px; margin:0 auto; font-size:14px; text-align:center; width:80%; line-height:16px; background: transparent; color:#636e72;">{{word}}</div>
        <Button v-if="!SighinStatus" type="primary" icon="ios-alarm" @click="Sighin" long style="margin-top:20px; margin-bottom:20px; margin-left:10%; width:80%;">Điểm danh</Button>
        <Button v-else type="primary" icon="ios-alarm" long disabled style="margin-top:20px; margin-bottom:20px; margin-left:10%; width:80%;">
            Điểm danh
        </Button>
      </Panel>
      </Col>
      <!-- <Col :xs={span:0} :sm={span:0} :md={span:24}> -->
      <Col>
      <Panel shadow style="margin-top: 37px;padding-bottom: 5px;">
        <div slot="title" style="margin-left: -10px;margin-bottom: -10px;">{{$t('m.Similar_Site')}}</div>
        <ul style="margin-left: 40px;margin-bottom: 20px;">
          <li style="padding: 5px 0px;"><a href="#" class="link-style" onclick="event.preventDefault();window.open('https://ispace.edu.vn/?ref=ispace', '_blank');">Trường Cao đẳng An ninh mạng iSPACE (🇻🇳)</a></li>
          <li style="padding: 5px 0px;"><a href="#" class="link-style" onclick="event.preventDefault();window.open('https://freecontest.net/?ref=ispace', '_blank');">Free Contest (🇻🇳)</a></li>
          <li style="padding: 5px 0px;"><a href="#" class="link-style" onclick="event.preventDefault();window.open('https://oj.vnoi.info/?ref=ispace', '_blank');">VNOJ: VNOI Online Judge (🇻🇳)</a></li>
          <li style="padding: 5px 0px;"><a href="#" class="link-style" onclick="event.preventDefault();window.open('http://ntucoder.net/?ref=ispace', '_blank');">NTUCoder (🇻🇳)</a></li>
          <li style="padding: 5px 0px;"><a href="#" class="link-style" onclick="event.preventDefault();window.open('https://codeforces.com/?ref=ispace', '_blank');">Codeforces (🇬🇧)</a></li>
          <li style="padding: 5px 0px;"><a href="#" class="link-style" onclick="event.preventDefault();window.open('https://leetcode.com/?ref=ispace', '_blank');">LeetCode (🇬🇧)</a></li>
        </ul>
      </Panel>
      </Col>
    </Col>
  </Row>
</template>

<script>
  import api from '@oj/api'
  import Pagination from '@oj/components/Pagination'
  import { mapState } from 'vuex'

  export default {
    name: 'Announcement',
    components: {
      Pagination
    },
    data () {
      return {
        limit: 10,
        total: 10,
        tagList: [],
        problemList: [],
        problemLimit: 15,
        query: {
          keyword: '',
          difficulty: '',
          tag: '',
          page: 1,
          orderby: '-create_time'
        },
        btnLoading: false,
        announcements: [],
        announcement: '',
        listVisible: true,
        timer: null,
        containerSpan: 18,
        SighinStatus: false,
        nowWeek: '',
        nowDate: '',
        word: '',
        days: 0
      }
    },
    mounted () {
      this.init()
      this.timer = setInterval(() => {
        this.setNowTimes()
      }, 1000)
      this.getWord()
      this.isSighin()
      this.getProblemList()
      this.getTagList()
    },
    methods: {
      init () {
        if (this.isContest) {
          this.containerSpan = 24
          this.getContestAnnouncementList()
        } else {
          this.getAnnouncementList()
        }
      },
      getTagList () {
        api.getProblemTagList().then(res => {
          this.tagList = res.data.data.sort((a, b) => {
            return a.id - b.id
          })
        })
      },
      getProblemList () {
        let offset = 0
        api.getProblemList(offset, this.problemLimit, this.query).then(res => {
          this.problemList = res.data.data.results
        })
      },
      getAnnouncementList (page = 1) {
        this.btnLoading = true
        api.getAnnouncementList((page - 1) * this.limit, this.limit).then(res => {
          this.btnLoading = false
          this.announcements = res.data.data.results
          this.total = res.data.data.total
        }, () => {
          this.btnLoading = false
        })
      },
      getContestAnnouncementList () {
        this.btnLoading = true
        api.getContestAnnouncementList(this.$route.params.contestID).then(res => {
          this.btnLoading = false
          this.announcements = res.data.data
        }, () => {
          this.btnLoading = false
        })
      },
      goAnnouncement (announcement) {
        this.announcement = announcement
        this.listVisible = false
      },
      goBack () {
        this.listVisible = true
        this.announcement = ''
      },
      getWord () {
        // axios.get('https://v1.hitokoto.cn/?c=d&c=e&c=f&c=h&c=i&c=j&c=k').then(response => {
        //   this.word = response.data.hitokoto
        // })
        this.word = '- Learning by Doing -'
      },
      setNowTimes () {
        let myDate = new Date()
        let weeks = ['Chủ nhật', 'Thứ 2', 'Thứ 3', 'Thứ 4', 'Thứ 5', 'Thứ 6', 'Thứ 7']
        let mouth = ['01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12']
        this.nowDate = String(myDate.getDate() < 10 ? '0' + myDate.getDate() : myDate.getDate()) + '/' + mouth[myDate.getMonth()] + '/' + myDate.getFullYear()
        this.nowWeek = weeks[myDate.getDay()]
      },
      isSighin () {
        api.GetSighinStatus().then(res => {
          if (res.data.data === null || res.data.data.sighinstatus === 'false') {
            this.SighinStatus = false
          } else {
            this.SighinStatus = true
          }
          this.days = res.data.data !== null ? res.data.data.continue_sighin_days : 0
        })
      },
      Sighin () {
        api.UserSighin().then(res => {
          if (res.data.data === 'Singined') {
            this.$Notice.error({
              title: 'Ghi danh không thành công',
              desc: 'Bạn ghi danh hôm nay rồi, hãy quay lại vào ngày mai nhé'
            })
            this.isSighin()
          } else {
            this.$Notice.success({
              title: 'Ghi danh thành công',
              desc: 'Chúc mừng bạn có thêm ' + (res.data.data.experience) + ' điểm kinh nghiệm, nhớ quay lại vào ngày mai nhé.'
            })
            this.days += 1
            this.SighinStatus = true
          }
        })
      }
    },
    computed: {
      ...mapState(['website']),
      title () {
        if (this.listVisible) {
          return this.isContest ? this.$i18n.t('m.Contest_Announcements') : this.$i18n.t('m.Announcements')
        } else {
          return this.announcement.title
        }
      },
      isContest () {
        return !!this.$route.params.contestID
      }
    }
  }
</script>

<style scoped lang="less">
  .announcements-container {
    margin-top: -10px;
    margin-bottom: 10px;
    li {
      padding-top: 15px;
      list-style: none;
      padding-bottom: 15px;
      margin-left: 20px;
      font-size: 16px;
      border-bottom: 1px solid rgba(187, 187, 187, 0.5);
      &:last-child {
        border-bottom: none;
      }
      .flex-container {
        .title {
          flex: 1 1;
          text-align: left;
          padding-left: 10px;
          a.entry {
            color: #495060;
            &:hover {
              color: #fb8500;
              border-bottom: 1px solid #fb8500;
            }
          }
        }
        .creator {
          flex: none;
          width: 200px;
          text-align: center;
        }
        .date {
          flex: none;
          width: 200px;
          text-align: center;
        }
      }
    }
  }

  .content-container {
    padding: 0 20px 20px 20px;
  }

  .no-announcement {
    text-align: center;
    font-size: 16px;
  }changeLocale

  .announcement-animate-enter-active {
    color: rgba(255, 255, 255, 0.5);
    animation: fadeIn 1s;
  }
  .nowWeek {
    text-align: center;
    padding-top: 20px;
    font-size: 25px;
    font-weight: 600;
  }
  .nowDate {
    text-align: center;
    padding-bottom: 10px;
    font-size: 30px;
    font-weight: 600;
  }
  .tag-btn {
    margin-right: 10px;
    margin-bottom: 20px;
  }
  .tag-btn:hover {
    border-color: #fb8500;
    a {
       // color: #2d8cf0;
       color: #fb8500;
    }
  }
  .link-style {
    color:#264c67
  }
  .link-style:hover {
    // color: #2d8cf0;
    color: #fb8500;
  }
</style>
