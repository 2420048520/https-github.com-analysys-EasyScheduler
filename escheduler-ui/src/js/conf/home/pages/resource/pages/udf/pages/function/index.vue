<template>
  <m-list-construction :title="$t('UDF Function')">
    <template slot="conditions">
      <m-conditions @on-conditions="_onConditions">
        <template slot="button-group">
          <x-button type="ghost" @click="_create" v-ps="['GENERAL_USER']" size="small" >{{$t('Create UDF Function')}}</x-button>
        </template>
      </m-conditions>
    </template>
    <template slot="content">
      <template v-if="udfFuncList.length">
        <m-list :udf-func-list="udfFuncList" :page-no="searchParams.pageNo" :page-size="searchParams.pageSize" @on-update="_updateList">
        </m-list>
        <div class="page-box">
          <x-page :current="parseInt(searchParams.pageNo)" :total="total" :page-size="searchParams.pageSize" show-elevator @on-change="_page"></x-page>
        </div>
      </template>
      <template v-if="!udfFuncList.length">
        <m-no-data></m-no-data>
      </template>
      <m-spin :is-spin="isLoading">
      </m-spin>
    </template>
  </m-list-construction>
</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import mList from './_source/list'
  import mCreateUdf from './_source/createUdf'
  import mSpin from '@/module/components/spin/spin'
  import mNoData from '@/module/components/noData/noData'
  import listUrlParamHandle from '@/module/mixin/listUrlParamHandle'
  import mConditions from '@/module/components/conditions/conditions'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  export default {
    name: 'udf-function-index',
    data () {
      return {
        total: null,
        isLoading: false,
        udfFuncList: [],
        searchParams: {
          pageSize: 10,
          pageNo: 1,
          searchVal: ''
        }
      }
    },
    mixins: [listUrlParamHandle],
    props: {},
    methods: {
      ...mapActions('resource', ['getUdfFuncListP']),
      _onConditions (o) {
        this.searchParams = _.assign(this.searchParams, o)
        this.searchParams.pageNo = 1
      },
      _page (val) {
        this.searchParams.pageNo = val
      },
      _create () {
        let self = this
        let modal = this.$modal.dialog({
          closable: false,
          showMask: true,
          escClose: true,
          className: 'v-modal-custom',
          transitionName: 'opacityp',
          render (h) {
            return h(mCreateUdf, {
              on: {
                onUpdate () {
                  self._updateList()
                  modal.remove()
                },
                close () {
                  modal.remove()
                }
              },
              props: {
              }
            })
          }
        })
      },
      _updateList () {
        this.searchParams.pageNo = 1
        this.searchParams.searchVal = ''
        this._debounceGET()
      },
      _getList (flag) {
        this.isLoading = !flag
        this.getUdfFuncListP(this.searchParams).then(res => {
          this.udfFuncList = []
          this.udfFuncList = res.totalList
          this.total = res.total
          this.isLoading = false
        }).catch(e => {
          this.isLoading = false
        })
      }
    },
    watch: {
      // router
      '$route' (a) {
        // url no params get instance list
        this.searchParams.pageNo = _.isEmpty(a.query) ? 1 : a.query.pageNo
      }
    },
    created () {
    },
    mounted () {
    },
    components: { mListConstruction, mConditions, mList, mSpin, mCreateUdf, mNoData }
  }
</script>
