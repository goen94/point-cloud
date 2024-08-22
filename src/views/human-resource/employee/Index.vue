<template>
  <div>
    <breadcrumb>
      <breadcrumb-human-resource />
      <span class="breadcrumb-item active">{{ 'employee' | uppercase }}</span>
    </breadcrumb>

    <tab-menu />

    <div class="row">
      <p-block>
        <div class="input-group block mb-5">
          <a
            v-if="$permission.has('create employee')"
            class="input-group-prepend"
            href="javascript:void(0)"
            @click="$refs.addEmployee.open()"
          >
            <span class="input-group-text">
              <i class="fa fa-plus" />
            </span>
          </a>
          <p-form-input
            id="search-text"
            ref="searchText"
            :value="searchText"
            class="btn-block"
            name="search-text"
            placeholder="Search"
            @input="filterSearch"
          />
        </div>
        <hr>
        <div class="text-center font-size-sm mb-10">
          <a
            href="javascript:void(0)"
            @click="isAdvanceFilter = !isAdvanceFilter"
          >
            {{ $t('advance filter') | uppercase }} <i class="fa fa-caret-down" />
          </a>
        </div>
        <div
          v-show="isAdvanceFilter"
          :class="{ 'fadeIn': isAdvanceFilter }"
          class="card"
        >
          <div class="row">
            <div class="col-sm-3 text-center">
              <p-form-row
                id="status"
                :is-horizontal="false"
                :label="$t('status')"
                name="status"
              >
                <div slot="body">
                  <span
                    class="select-link"
                    @click="$refs.status.open({ id: statusId, label: statusLabel })"
                  >
                    {{ statusLabel || $t('select') | uppercase }}
                  </span>
                </div>
              </p-form-row>
            </div>
          </div>
        </div>
        <hr>
        <div
          v-show="checkedRow.length > 0"
          class="mr-15 animated fadeIn"
        >
          <button
            class="btn btn-secondary mr-5"
            type="button"
            @click="bulkArchiveEmployee()"
          >
            {{ $t('archive') | uppercase }}
          </button>
          <button
            class="btn btn-secondary mr-5"
            type="button"
            @click="bulkActivateEmployee()"
          >
            {{ $t('activate') | uppercase }}
          </button>
          <button
            class="btn btn-secondary"
            type="button"
            @click="bulkDeleteEmployee()"
          >
            {{ $t('delete') | uppercase }}
          </button>
        </div>
        <p-block-inner :is-loading="isLoading">
          <point-table>
            <tr slot="p-head">
              <th width="50px">
                #
              </th>
              <th width="50px">
                <p-form-check-box
                  id="subscibe"
                  :checked="isRowsChecked(employees, checkedRow)"
                  :is-form="false"
                  class="text-center"
                  name="subscibe"
                  @click.native="toggleCheckRows()"
                />
              </th>
              <th>{{ $t('name') }}</th>
              <th>{{ $t('job title') }}</th>
              <th>{{ $t('department') }}</th>
              <th>{{ $t('status contract') }}</th>
            </tr>
            <template v-for="(employee, index) in employees">
              <tr
                v-if="($permission.has('create employee assessment') && isShow(employee.scorers)) || $permission.has('read employee')"
                :key="employee.id"
                slot="p-body"
                :class="{
                  'bg-gray': employee.archived_at != null,
                  'bg-primary-lighter': isRowChecked(employee.id)
                }"
              >
                <th
                  :class="{
                    'bg-gray': employee.archived_at != null,
                    'bg-primary-lighter': isRowChecked(employee.id)
                  }"
                >
                  {{ getNumberIndex(index) }}
                </th>
                <td>
                  <p-form-check-box
                    id="subscibe"
                    :checked="isRowChecked(employee.id)"
                    :is-form="false"
                    class="text-center"
                    name="subscibe"
                    @click.native="toggleCheckRow(employee.id)"
                  />
                </td>
                <td>
                  <router-link :to="{ name: 'EmployeeShow', params: { id: employee.id }}">
                    {{ employee.name }}
                  </router-link>
                </td>
                <td>{{ employee.job_title }}</td>
                <td>
                  <template v-if="employee.group">
                    {{ employee.group.name }}
                  </template>
                </td>
                <td>
                  <template v-if="employee.status">
                    <span>{{ employee.status.name }}</span>
                  </template>
                </td>
              </tr>
            </template>
          </point-table>
        </p-block-inner>
        <p-pagination
          :current-page="page"
          :last-page="lastPage"
          @updatePage="updatePage"
        />
      </p-block>
    </div>
    <m-add-employee
      ref="addEmployee"
      @added="onAdded"
    />
    <m-status
      ref="status"
      @choosen="onChoosenStatus"
    />
  </div>
</template>

<script>
import TabMenu from '@/views/human-resource/TabMenu'

import Breadcrumb from '@/views/Breadcrumb'
import BreadcrumbHumanResource from '@/views/human-resource/Breadcrumb'
import PointTable from 'point-table-vue'
import debounce from 'lodash/debounce'
import { mapActions, mapGetters } from 'vuex'

export default {
  components: {
    TabMenu,
    Breadcrumb,
    BreadcrumbHumanResource,
    PointTable
  },
  data () {
    const statusId = parseInt(this.$route.query.statusId, 10)
    return {
      isLoading: false,
      searchText: this.$route.query.search,
      page: this.$route.query.page * 1 || 1,
      lastPage: 1,
      isAdvanceFilter: false,
      checkedRow: [],
      statusId: statusId >= 1 && statusId <= 2 ? statusId : null,
      statusLabel: null
    }
  },
  computed: {
    ...mapGetters('auth', ['authUser']),
    ...mapGetters('humanResourceEmployee', ['employees', 'pagination']),
    ...mapGetters('humanResourceEmployeeGroup', ['groupList'])
  },
  created () {
    this.getEmployeesRequest()
  },
  updated () {
    this.lastPage = this.pagination.last_page
  },
  methods: {
    ...mapActions('humanResourceEmployee', {
      getEmployees: 'get',
      bulkArchive: 'bulkArchive',
      bulkActivate: 'bulkActivate',
      bulkDelete: 'bulkDelete'
    }),
    getNumberIndex (index) {
      return (this.page * 10) - 10 + index + 1
    },
    onAdded () {
      this.getEmployeesRequest()
    },
    toggleCheckRow (id) {
      if (!this.isRowChecked(id)) {
        this.checkedRow.push({ id })
      } else {
        this.checkedRow.splice(this.checkedRow.map((o) => o.id).indexOf(id), 1)
      }
    },
    toggleCheckRows () {
      if (!this.isRowsChecked(this.employees, this.checkedRow)) {
        this.employees.forEach(element => {
          if (!this.isRowChecked(element.id)) {
            const id = element.id
            this.checkedRow.push({ id })
          }
        })
      } else {
        this.employees.forEach(element => {
          this.checkedRow.splice(this.checkedRow.map((o) => o.id).indexOf(element.id), 1)
        })
      }
    },
    isRowChecked (id) {
      return this.checkedRow.some(element => {
        return element.id == id
      })
    },
    isRowsChecked (haystack, needles) {
      if (needles.length == 0) {
        return false
      }
      for (let i = 0; i < haystack.length; i++) {
        const found = needles.some(element => {
          return element.id == haystack[i].id
        })
        if (!found) {
          return false
        }
      }
      return true
    },
    bulkArchiveEmployee () {
      this.bulkArchive({
        employees: this.checkedRow
      }).then(response => {
        this.checkedRow = []
        this.getEmployeesRequest()
      })
    },
    bulkActivateEmployee () {
      this.bulkActivate({
        employees: this.checkedRow
      }).then(response => {
        this.checkedRow = []
        this.getEmployeesRequest()
      })
    },
    bulkDeleteEmployee () {
      this.bulkDelete({
        employees: this.checkedRow
      }).then(response => {
        this.checkedRow = []
        this.getEmployeesRequest()
      })
    },
    isShow (scorers) {
      return scorers.some(element => {
        return element.id == this.authUser.id
      })
    },
    updatePage (value) {
      this.page = value
      this.getEmployeesRequest()
    },
    onChoosenStatus (option) {
      this.statusId = option.id
      this.statusLabel = option.label
      this.$router.push({
        query: {
          search: this.searchText,
          statusId: this.statusId
        }
      })
      this.getEmployeesRequest()
    },
    filterSearch: debounce(function (value) {
      this.$router.push({ query: { search: value } })
      this.searchText = value
      this.page = 1
      this.getEmployeesRequest()
    }, 300),
    getEmployeesRequest () {
      this.isLoading = true
      this.getEmployees({
        params: {
          filter_like: {
            name: this.searchText,
            job_title: this.searchText
          },
          limit: 10,
          page: this.page,
          status: this.statusId,
          sort_by: 'name',
          includes: 'scorers',
          additional: 'groups'
        }
      }).then((response) => {
        this.isLoading = false
      }, (errors) => {
        this.isLoading = false
        console.log(errors.data)
      })
    }
  }
}
</script>
