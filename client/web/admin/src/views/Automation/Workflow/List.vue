<template>
  <b-container
    fluid="xl"
  >
    <c-content-header
      :title="$t('title')"
    >
      <span
        class="text-nowrap"
      >
        <b-button
          v-if="canCreate"
          variant="primary"
          :to="{ name: 'automation.workflow.new' }"
        >
          {{ $t('new') }}
        </b-button>

        <c-permissions-button
          v-if="canGrant"
          resource="corteza::automation:workflow/*"
          button-variant="light"
          class="ml-2"
        >
          <font-awesome-icon :icon="['fas', 'lock']" />
          {{ $t('permissions') }}
        </c-permissions-button>
      </span>

      <b-dropdown
        v-if="false"
        variant="link"
        right
        menu-class="shadow-sm"
        :text="$t('export')"
      >
        <b-dropdown-item-button variant="link">
          {{ $t('yaml') }}
        </b-dropdown-item-button>
      </b-dropdown>
    </c-content-header>

    <c-resource-list
      :primary-key="primaryKey"
      :filter="filter"
      :sorting="sorting"
      :pagination="pagination"
      :fields="fields"
      :items="items"
      :row-class="genericRowClass"
      :translations="{
        searchPlaceholder: $t('filterForm.query.placeholder'),
        notFound: $t('admin:general.notFound'),
        noItems: $t('admin:general.resource-list.no-items'),
        loading: $t('loading'),
        showingPagination: 'admin:general.pagination.showing',
        singlePluralPagination: 'admin:general.pagination.single',
        prevPagination: $t('admin:general.pagination.prev'),
        nextPagination: $t('admin:general.pagination.next'),
        resourceSingle: $t('general:label.workflow.single'),
        resourcePlural: $t('general:label.workflow.plural')
      }"
      clickable
      sticky-header
      class="custom-resource-list-height"
      @search="filterList"
      @row-clicked="handleRowClicked"
    >
      <template #header>
        <c-resource-list-status-filter
          v-model="filter.deleted"
          :label="$t('filterForm.deleted.label')"
          :excluded-label="$t('filterForm.excluded.label')"
          :inclusive-label="$t('filterForm.inclusive.label')"
          :exclusive-label="$t('filterForm.exclusive.label')"
          @change="filterList"
        />
      </template>

      <template #actions="{ item: w }">
        <b-dropdown
          v-if="(areActionsVisible({ resource: w, conditions: ['canGrant', 'canDeleteWorkflow'] }) && w.workflowID)"
          boundary="viewport"
          variant="outline-light"
          toggle-class="d-flex align-items-center justify-content-center text-primary border-0 py-2"
          no-caret
          dropleft
          lazy
          menu-class="m-0"
        >
          <template #button-content>
            <font-awesome-icon
              :icon="['fas', 'ellipsis-v']"
            />
          </template>

          <b-dropdown-item
            v-if="w.workflowID && canGrant"
            link-class="p-0"
            variant="light"
          >
            <c-permissions-button
              :title="w.meta.name || w.handle || w.workflowID"
              :target="w.meta.name || w.handle || w.workflowID"
              :resource="`corteza::automation:workflow/${w.workflowID}`"
              button-variant="link dropdown-item text-decoration-none text-dark regular-font rounded-0"
            >
              <font-awesome-icon :icon="['fas', 'lock']" />
              {{ $t('permissions') }}
            </c-permissions-button>
          </b-dropdown-item>

          <c-input-confirm
            v-if="w.canDeleteWorkflow && !w.deletedAt"
            borderless
            variant="link"
            size="md"
            button-class="dropdown-item text-decoration-none text-dark regular-font rounded-0"
            class="w-100"
            @confirmed="handleDelete(w)"
          >
            <font-awesome-icon
              :icon="['far', 'trash-alt']"
              class="text-danger"
            />
            <span
              class="p-1"
            >
              {{ $t('delete') }}
            </span>
          </c-input-confirm>

          <c-input-confirm
            v-if="w.canUndeleteWorkflow && w.deletedAt"
            borderless
            variant="link"
            size="md"
            button-class="dropdown-item text-decoration-none text-dark regular-font rounded-0"
            class="w-100"
            @confirmed="handleDelete(w)"
          >
            <font-awesome-icon
              :icon="['far', 'trash-alt']"
              class="text-danger"
            />
            <span
              class="p-1"
            >
              {{ $t('undelete') }}
            </span>
          </c-input-confirm>
        </b-dropdown>
      </template>
    </c-resource-list>
  </b-container>
</template>

<script>
import * as moment from 'moment'
import listHelpers from 'corteza-webapp-admin/src/mixins/listHelpers'
import { mapGetters } from 'vuex'
import { components } from '@cortezaproject/corteza-vue'
const { CResourceList } = components

export default {
  components: {
    CResourceList,
  },

  mixins: [
    listHelpers,
  ],

  i18nOptions: {
    namespaces: 'automation.workflows',
    keyPrefix: 'list',
  },

  data () {
    return {
      id: 'workflow',

      primaryKey: 'workflowID',
      editRoute: 'automation.workflow.edit',

      filter: {
        query: '',
        deleted: 0,
        disabled: 1,
      },

      sorting: {
        sortBy: 'createdAt',
        sortDesc: true,
      },

      fields: [
        {
          key: 'handle',
          sortable: true,
        },
        {
          key: 'meta.name',
          label: this.$t(`columns.name`),
        },
        {
          key: 'enabled',
          sortable: true,
          formatter: (v) => v ? 'Yes' : 'No',
        },
        {
          key: 'createdAt',
          sortable: true,
          formatter: (v) => moment(v).fromNow(),
        },
        {
          key: 'actions',
          class: 'actions',
        },
      ].map(c => ({
        ...c,
        // Generate column label translation key
        label: c.label || this.$t(`columns.${c.key}`),
      })),
    }
  },

  computed: {
    ...mapGetters({
      can: 'rbac/can',
    }),

    canCreate () {
      return this.can('automation/', 'workflow.create')
    },

    canGrant () {
      return this.can('automation/', 'grant')
    },
  },

  methods: {
    items () {
      return this.procListResults(this.$AutomationAPI.workflowList(this.encodeListParams()))
    },

    handleDelete (workflow) {
      this.handleItemDelete({
        resource: workflow,
        resourceName: 'workflow',
        api: 'automation',
      })
    },
  },
}
</script>
