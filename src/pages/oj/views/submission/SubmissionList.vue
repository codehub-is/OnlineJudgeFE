<template>
  <!-- <div class="flex-container"> -->
    <Row type="flex" :gutter="18">
      <Col :span="containerSpan">
    <div id="main">
      <Panel shadow>
        <div slot="title">{{title}}</div>
        <div slot="extra" style="padding-top:30px;">
          <ul class="filter">
            <li>
              <Dropdown @on-click="handleResultChange">
                <span>{{status}}
                  <Icon type="md-arrow-dropdown"></Icon>
                </span>
                <Dropdown-menu slot="list">
                  <Dropdown-item name="">{{$t('m.All')}}</Dropdown-item>
                  <Dropdown-item v-for="status in Object.keys(JUDGE_STATUS)" :key="status" :name="status">
                    {{$t('m.' + JUDGE_STATUS[status].name.replace(/ /g, "_"))}}
                  </Dropdown-item>
                </Dropdown-menu>
              </Dropdown>
            </li>
            <li>
              <i-switch size="large" v-model="formFilter.myself" @on-change="handleQueryChange" style="min-width:70px;">
                <span slot="open">{{$t('m.Mine')}}</span>
                <span slot="close">{{$t('m.All')}}</span>
              </i-switch>
            </li>
            <li>
              <Input v-model="formFilter.username" :placeholder="$t('m.Search_Author')" @on-enter="handleQueryChange"/>
            </li>

            <li>
              <Button type="info" icon="md-refresh" @click="getSubmissions">{{$t('m.Refresh')}}</Button>
            </li>
          </ul>
        </div>
        <Table stripe :disabled-hover="true" :columns="columns" :data="submissions" :loading="loadingTable" style="margin-top:60px;"></Table>
        <Pagination :total="total" :page-size="limit" @on-change="changeRoute" :current.sync="page"></Pagination>
      </Panel>
    </div>
    </Col>

    <Col span="5" v-if="!contestID">
    <div v-if="!contestID" id="right-column">
      <Panel shadow style="padding-top: 0px;padding-bottom: 10px;min-height: 400px; min-width: 200px;">
        <div slot="title" style="margin-left: -10px;margin-bottom: -10px;">{{$t('m.Ranklist_Title')}}</div>
        <ol style="margin-left: 40px;margin-bottom: 20px;">
          <li v-for="u in dataRank" :key="u.id" style="margin-top:4px;">
            <a :style="'font-weight: 600;color: ' + u.color" :href="'/user-home?username=' + u.user.username"
               :title=" u.title + ' ' + u.user.username">
            {{u.user.username}}
            </a> - {{u.accepted_number}} bài
          </li>
        </ol>
      </Panel>
    </div>
    </Col>
  </Row>
  <!-- </div> -->
</template>

<script>
  import { mapGetters } from 'vuex'
  import api from '@oj/api'
  import { RULE_TYPE, JUDGE_STATUS, USER_TYPE, USER_GRADE } from '@/utils/constants'
  import utils from '@/utils/utils'
  import time from '@/utils/time'
  import Pagination from '@/pages/oj/components/Pagination'

  export default {
    name: 'submissionList',
    components: {
      Pagination
    },
    data () {
      return {
        containerSpan: 19,
        dataRank: [],
        rankLimit: 30,
        formFilter: {
          myself: false,
          result: '',
          username: ''
        },
        columns: [
          {
            title: this.$i18n.t('m.When'),
            align: 'center',
            render: (h, params) => {
              return h('span', time.utcToLocal(params.row.create_time))
            }
          },
          {
            title: this.$i18n.t('m.Author'),
            align: 'center',
            render: (h, params) => {
              if (params.row.title) {
                return h('a', {
                  style: {
                    'display': 'inline-block',
                    'margin-left': '5px',
                    'font-weight': 600,
                    'color': params.row.title_color
                  },
                  attrs: {
                    'title': params.row.title + ' ' + params.row.username,
                    'href': '/user-home?username=' + params.row.username
                  }
                }, params.row.username)
              } else {
                return h('a', {
                  style: {
                    'display': 'inline-block',
                    'max-width': '150px',
                    'margin-left': '5px',
                    'font-weight': 600,
                    'color': USER_GRADE[params.row.grade].color
                  },
                  attrs: {
                    'title': USER_GRADE[params.row.grade].name + ' ' + params.row.username,
                    'href': '/user-home?username=' + params.row.username
                  }
                }, params.row.username)
              }
            }
          },
          {
            title: this.$i18n.t('m.Status'),
            align: 'center',
            render: (h, params) => {
              return h('Tag', {
                props: {
                  color: JUDGE_STATUS[params.row.result].type
                }
              }, this.$i18n.t('m.' + JUDGE_STATUS[params.row.result].name.replace(/ /g, '_')))
            }
          },
          {
            title: this.$i18n.t('m.Problem'),
            align: 'center',
            render: (h, params) => {
              return h('a',
                {
                  style: {
                    color: '#57a3f3',
                    cursor: 'pointer'
                  },
                  attrs: {
                    'href': this.contestID ? '/contest/' + this.contestID + '/problem/' + params.row.problem : '/problem/' + params.row.problem
                  }
                },
                params.row.problem)
            }
          },
          {
            title: this.$i18n.t('m.Time'),
            align: 'center',
            render: (h, params) => {
              return h('span', utils.submissionTimeFormat(params.row.statistic_info.time_cost))
            }
          },
          {
            title: this.$i18n.t('m.Memory'),
            align: 'center',
            render: (h, params) => {
              return h('span', utils.submissionMemoryFormat(params.row.statistic_info.memory_cost))
            }
          },
          {
            title: this.$i18n.t('m.Language'),
            align: 'center',
            key: 'language'
          },
          {
            title: this.$i18n.t('m.ID'),
            align: 'center',
            render: (h, params) => {
              if (params.row.show_link) {
                return h('a', {
                  style: {
                    color: '#57a3f3',
                    cursor: 'pointer'
                  },
                  attrs: {
                    'href': '/status/' + params.row.id + '?problem=' + params.row.problem
                  }
                }, params.row.id.slice(0, 12))
              } else {
                return h('span', params.row.id.slice(0, 12))
              }
            }
          }
        ],
        loadingTable: false,
        submissions: [],
        total: 30,
        limit: 15,
        page: 1,
        contestID: '',
        problemID: '',
        routeName: '',
        JUDGE_STATUS: '',
        rejudge_column: false
      }
    },
    mounted () {
      this.init()
      this.getRankData()
      this.JUDGE_STATUS = Object.assign({}, JUDGE_STATUS)
      // 去除submitting的状态 和 两个
      delete this.JUDGE_STATUS['9']
      delete this.JUDGE_STATUS['2']
    },
    methods: {
      init () {
        this.contestID = this.$route.params.contestID
        let query = this.$route.query
        this.problemID = query.problemID
        this.formFilter.myself = query.myself === '1'
        this.formFilter.result = query.result || ''
        this.formFilter.username = query.username || ''
        this.page = parseInt(query.page) || 1
        if (this.page < 1) {
          this.page = 1
        }
        this.routeName = this.$route.name
        this.getSubmissions()
        if (this.contestID) {
          this.containerSpan = 24
        }
      },
      getRankData () {
        api.getUserRank(0, this.rankLimit, RULE_TYPE.ACM).then(res => {
          this.dataRank = res.data.data.results
          for (let i in this.dataRank) {
            this.dataRank[i]['color'] = USER_GRADE[this.dataRank[i].grade].color
            this.dataRank[i]['title'] = USER_GRADE[this.dataRank[i].grade].name
          }
        }).catch(() => {
        })
      },
      buildQuery () {
        return {
          myself: this.formFilter.myself === true ? '1' : '0',
          result: this.formFilter.result,
          username: this.formFilter.username,
          page: this.page
        }
      },
      getSubmissions () {
        let params = this.buildQuery()
        params.contest_id = this.contestID
        params.problem_id = this.problemID
        let offset = (this.page - 1) * this.limit
        let func = this.contestID ? 'getContestSubmissionList' : 'getSubmissionList'
        this.loadingTable = true
        api[func](offset, this.limit, params).then(res => {
          let data = res.data.data
          for (let v of data.results) {
            v.loading = false
          }
          this.adjustRejudgeColumn()
          this.loadingTable = false
          this.submissions = data.results
          this.total = data.total
        }).catch(() => {
          this.loadingTable = false
        })
      },
      // 改变route， 通过监听route变化请求数据，这样可以产生route history， 用户返回时就会保存之前的状态
      changeRoute () {
        let query = utils.filterEmptyValue(this.buildQuery())
        query.contestID = this.contestID
        query.problemID = this.problemID
        let routeName = query.contestID ? 'contest-submission-list' : 'submission-list'
        this.$router.push({
          name: routeName,
          query: utils.filterEmptyValue(query)
        })
      },
      goRoute (route) {
        this.$router.push(route)
      },
      adjustRejudgeColumn () {
        if (!this.rejudgeColumnVisible || this.rejudge_column) {
          return
        }
        const judgeColumn = {
          title: this.$i18n.t('m.Option'),
          fixed: 'right',
          align: 'center',
          width: 90,
          render: (h, params) => {
            return h('Button', {
              props: {
                type: 'primary',
                size: 'small',
                loading: params.row.loading
              },
              on: {
                click: () => {
                  this.handleRejudge(params.row.id, params.index)
                }
              }
            }, this.$i18n.t('m.Rejudge'))
          }
        }
        this.columns.push(judgeColumn)
        this.rejudge_column = true
      },
      handleResultChange (status) {
        this.page = 1
        this.formFilter.result = status
        this.changeRoute()
      },
      handleQueryChange () {
        this.page = 1
        this.changeRoute()
      },
      handleRejudge (id, index) {
        this.submissions[index].loading = true
        api.submissionRejudge(id).then(res => {
          this.submissions[index].loading = false
          this.$success('Succeeded')
          this.getSubmissions()
        }, () => {
          this.submissions[index].loading = false
        })
      }
    },
    computed: {
      ...mapGetters(['isAuthenticated', 'user']),
      title () {
        if (!this.contestID) {
          return this.$i18n.t('m.StatusTitle')
        } else if (this.problemID) {
          return this.$i18n.t('m.Problem_Submissions')
        } else {
          return this.$i18n.t('m.Submissions')
        }
      },
      status () {
        return this.formFilter.result === '' ? this.$i18n.t('m.Status') : this.$i18n.t('m.' + JUDGE_STATUS[this.formFilter.result].name.replace(/ /g, '_'))
      },
      rejudgeColumnVisible () {
        return !this.contestID && this.user.admin_type === USER_TYPE.SUPER_ADMIN
      }
    },
    watch: {
      '$route' (newVal, oldVal) {
        if (newVal !== oldVal) {
          this.init()
        }
      },
      'rejudgeColumnVisible' () {
        this.adjustRejudgeColumn()
      },
      'isAuthenticated' () {
        this.init()
      }
    }
  }
</script>

<style scoped lang="less">
  .ivu-btn-text {
    color: #57a3f3;
  }

  .flex-container {
    #main {
      flex: auto;
      margin-right: 18px;
      .filter {
        margin-right: -10px;
      }
    }
    #right-column {
      flex: none;
      width: 300px;
      max-width: 300px;
    }
  }
</style>
