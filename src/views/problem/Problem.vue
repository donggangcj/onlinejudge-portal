<template>
  <div>
    <section class="section-tertiary">
      <div class="container">
        <table>
          <thead>
            <tr>
              <th>#</th>
              <th>标题</th>
              <th>更新时间</th>
              <th>提交/通过比例</th>
              <th>操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="problem in formattedProblems" :key="problem.problemId">
              <td>{{problem.problemId}}</td>
              <td>{{problem.title}}</td>
              <td>{{problem.updateTime | timeAgo}}</td>
              <td>
                <progress-bar :percentage="problem.passRate"></progress-bar>
              </td>
              <td>
                <button class="button-primary-text" @click="readMore(problem.problemId)">More</button>
              </td>
            </tr>
          </tbody>
        </table>
        <my-pagination :total="pagination.total" :page="pagination.page" :rpp="pagination.rpp" @size-change="handleSizeChange" @current-change="handleCurrentPageChange"></my-pagination>
      </div>
    </section>
  </div>
</template>

<script>
import MyPagination from '@/components/pagination/MyPagination'
import ProgressBar from '@/components/progress-bar/ProgressBar'

import problemApi from '@/apis/problem'
import * as filterUtil from '@/utils/filter'

export default {
  components: {
    MyPagination,
    ProgressBar
  },
  data() {
    return {
      problems: [],
      pagination: {
        total: 0,
        page: 1,
        rpp: 10,
        sort: 'updateTime,desc'
      }
    }
  },
  computed: {
    formattedProblems() {
      const problems = this.problems.slice(0, this.problems.length)
      this.problems.slice(0, this.problems.length).forEach(problem => {
        problem.passRate = filterUtil.toPercentage(
          problem.totalPass,
          problem.totalSubmit
        )
      })
      return problems
    }
  },
  mounted() {
    this.getProblems()
  },
  methods: {
    getProblems() {
      problemApi
        .getProblems(
          this.pagination.page,
          this.pagination.rpp,
          this.pagination.sort
        )
        .then(response => {
          if (response && response.status === 200) {
            this.problems = response.data.datas
            this.pagination.total = response.data.total
          }
        })
    },
    readMore(id) {
      this.$router.push({ name: 'problem_detail', params: { problemId: id } })
    },
    handleSizeChange(rpp) {
      this.pagination.rpp = rpp
      this.getProblems()
    },
    handleCurrentPageChange(currentPage) {
      this.pagination.page = currentPage
      this.getProblems()
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
