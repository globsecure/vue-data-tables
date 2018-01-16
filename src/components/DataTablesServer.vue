<style lang='scss'>
  @import "../style/index.scss";
</style>

<script>
  import ShareMixin from '../mixins/ShareMixin'
  import debounce from 'javascript-debounce'

  export default {
    name: 'DataTablesServer',
    mixins: [ShareMixin],
    props: {
      loadingStr: {
        type: String,
        default: ''
      },
      total: {
        type: Number
      },
      reset: {
        type: Boolean,
        default: false
      },
      loadData: {
        type: Function
      },
      loading: {
        type: Boolean,
        default: false
      }
    },
    created() {
      this._server = true
      this.queryChange('init')
    },
    data() {
      return {
        timeout: 0,
        innerLoading: false
      }
    },
    computed: {
      curTableData() {
        return this.data.length > this.innerPageSize
          ? this.data.slice(0, this.innerPageSize)
          : this.data
      },
      queryInfo() {
        return {
          page: this.currentPage,
          pageSize: this.innerPageSize,
          sortInfo: this.sortData,
          filters: this.filters
        }
      },
      updateInnerSearchKey() {
        const timeout = this.innerSearchDef.debounceTime
        return debounce(_ => {
          this.innerSearchKey = this.searchKey
          this.queryChange('searchBoxChange')
        }, timeout)
      }
    },
    methods: {
      resetFilter() {
        this.queryInfo.filters.map(filter => {
          filter.vals = ''
        })
        let info = {
          type: 'customFilterChange',
          ...this.queryInfo
        }
        this.loadData && this.innerLoadData(info)
      },
      queryChange(type) {
        clearTimeout(this.timeout)
        this.timeout = setTimeout(() => {
          let info = {
            type,
            ...this.queryInfo
          }
          this.loadData && this.innerLoadData(info)
        }, this.innerSearchDef.debounceTime)
      },
      handleSizeChange(size) {
        this.innerPageSize = size
        this.queryChange('sizeChange')
        this.$emit('size-change', size)
      },
      handlePageChange(currentPage) {
        this.currentPage = currentPage
        this.queryChange('pageChange')
        this.$emit('current-change', currentPage)
      },
      handleCheckBoxValChange(checkBoxValues) {
        this.checkBoxValues = checkBoxValues
        this.queryChange('checkBoxChange')
      },
      handleSort(obj) {
        if (this.sortData.prop === obj.prop && this.sortData.order === obj.order) {
          this.sortData = obj
        } else {
          this.sortData = obj
          this.queryChange('sortChange')
        }
      },
      innerLoadData(info) {
        let autoChange = (info.type === 'pageChange' || info.type === 'init' || info.type === 'sortChange' || info.type === 'checkBoxChange' || info.type === 'sizeChange')
        if ((info.filters.length >= 1 && info.type === 'customFilterChange') || autoChange) {
          this.innerLoading = true
          this.loadData && this.loadData(info)
            .then(data => {
              this.innerLoading = false
              this.$emit('load-data-success', data, info)
            })
            .catch(error => {
              this.innerLoading = false
              this.$emit('load-data-fail', error, info)
            })
        } else {
          console.info('innerLoadData', 'Call not executed, parameters are not enough.')
        }
      }
    },
    watch: {
      innerCustomFilters() {
        this.queryChange('customFilterChange')
      },
      reset() {
        this.resetFilter()
      },
      loading(val) {
        this.innerLoading = val
      },
      'tableProps.defaultSort': {
        immediate: true,
        handler(val) {
          this.sortData = val || {}
        }
      }
    }
  }
</script>
