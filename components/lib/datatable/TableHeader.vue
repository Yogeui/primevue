<template>
    <thead class="p-datatable-thead" role="rowgroup" v-bind="ptm('thead')">
        <template v-if="!columnGroup">
            <tr role="row" v-bind="ptm('headerRow')">
                <template v-for="(col, i) of columns" :key="columnProp(col, 'columnKey') || columnProp(col, 'field') || i">
                    <DTHeaderCell
                        v-if="!columnProp(col, 'hidden') && (rowGroupMode !== 'subheader' || groupRowsBy !== columnProp(col, 'field'))"
                        :column="col"
                        @column-click="$emit('column-click', $event)"
                        @column-mousedown="$emit('column-mousedown', $event)"
                        @column-dragstart="$emit('column-dragstart', $event)"
                        @column-dragover="$emit('column-dragover', $event)"
                        @column-dragleave="$emit('column-dragleave', $event)"
                        @column-drop="$emit('column-drop', $event)"
                        :groupRowsBy="groupRowsBy"
                        :groupRowSortField="groupRowSortField"
                        :reorderableColumns="reorderableColumns"
                        :resizableColumns="resizableColumns"
                        @column-resizestart="$emit('column-resizestart', $event)"
                        :sortMode="sortMode"
                        :sortField="sortField"
                        :sortOrder="sortOrder"
                        :multiSortMeta="multiSortMeta"
                        :allRowsSelected="allRowsSelected"
                        :empty="empty"
                        @checkbox-change="$emit('checkbox-change', $event)"
                        :filters="filters"
                        :filterDisplay="filterDisplay"
                        :filtersStore="filtersStore"
                        :filterInputProps="filterInputProps"
                        @filter-change="$emit('filter-change', $event)"
                        @filter-apply="$emit('filter-apply')"
                        @operator-change="$emit('operator-change', $event)"
                        @matchmode-change="$emit('matchmode-change', $event)"
                        @constraint-add="$emit('constraint-add', $event)"
                        @constraint-remove="$emit('constraint-remove', $event)"
                        @apply-click="$emit('apply-click', $event)"
                        :headerCheckboxIconTemplate="headerCheckboxIconTemplate"
                        :pt="pt"
                    />
                </template>
            </tr>
            <tr v-if="filterDisplay === 'row'" role="row" v-bind="ptm('headerRow')">
                <template v-for="(col, i) of columns" :key="columnProp(col, 'columnKey') || columnProp(col, 'field') || i">
                    <th v-if="!columnProp(col, 'hidden') && (rowGroupMode !== 'subheader' || groupRowsBy !== columnProp(col, 'field'))" :style="getFilterColumnHeaderStyle(col)" :class="getFilterColumnHeaderClass(col)" v-bind="ptm('headerCell')">
                        <DTHeaderCheckbox v-if="columnProp(col, 'selectionMode') === 'multiple'" :checked="allRowsSelected" :disabled="empty" @change="$emit('checkbox-change', $event)" :pt="pt" />
                        <DTColumnFilter
                            v-if="col.children && col.children.filter"
                            :field="columnProp(col, 'filterField') || columnProp(col, 'field')"
                            :type="columnProp(col, 'dataType')"
                            display="row"
                            :showMenu="columnProp(col, 'showFilterMenu')"
                            :filterElement="col.children && col.children.filter"
                            :filterHeaderTemplate="col.children && col.children.filterheader"
                            :filterFooterTemplate="col.children && col.children.filterfooter"
                            :filterClearTemplate="col.children && col.children.filterclear"
                            :filterApplyTemplate="col.children && col.children.filterapply"
                            :filterIconTemplate="col.children && col.children.filtericon"
                            :filterAddIconTemplate="col.children && col.children.filteraddicon"
                            :filterRemoveIconTemplate="col.children && col.children.filterremoveicon"
                            :filterClearIconTemplate="col.children && col.children.filterclearicon"
                            :filters="filters"
                            :filtersStore="filtersStore"
                            :filterInputProps="filterInputProps"
                            @filter-change="$emit('filter-change', $event)"
                            @filter-apply="$emit('filter-apply')"
                            :filterMenuStyle="columnProp(col, 'filterMenuStyle')"
                            :filterMenuClass="columnProp(col, 'filterMenuClass')"
                            :showOperator="columnProp(col, 'showFilterOperator')"
                            :showClearButton="columnProp(col, 'showClearButton')"
                            :showApplyButton="columnProp(col, 'showApplyButton')"
                            :showMatchModes="columnProp(col, 'showFilterMatchModes')"
                            :showAddButton="columnProp(col, 'showAddButton')"
                            :matchModeOptions="columnProp(col, 'filterMatchModeOptions')"
                            :maxConstraints="columnProp(col, 'maxConstraints')"
                            @operator-change="$emit('operator-change', $event)"
                            @matchmode-change="$emit('matchmode-change', $event)"
                            @constraint-add="$emit('constraint-add', $event)"
                            @constraint-remove="$emit('constraint-remove', $event)"
                            @apply-click="$emit('apply-click', $event)"
                            :pt="pt"
                        />
                    </th>
                </template>
            </tr>
        </template>
        <template v-else>
            <tr v-for="(row, i) of getHeaderRows()" :key="i" role="row" v-bind="ptm('headerRow')">
                <template v-for="(col, j) of getHeaderColumns(row)" :key="columnProp(col, 'columnKey') || columnProp(col, 'field') || j">
                    <DTHeaderCell
                        v-if="!columnProp(col, 'hidden') && (rowGroupMode !== 'subheader' || groupRowsBy !== columnProp(col, 'field')) && typeof col.children !== 'string'"
                        :column="col"
                        @column-click="$emit('column-click', $event)"
                        @column-mousedown="$emit('column-mousedown', $event)"
                        :groupRowsBy="groupRowsBy"
                        :groupRowSortField="groupRowSortField"
                        :sortMode="sortMode"
                        :sortField="sortField"
                        :sortOrder="sortOrder"
                        :multiSortMeta="multiSortMeta"
                        :allRowsSelected="allRowsSelected"
                        :empty="empty"
                        @checkbox-change="$emit('checkbox-change', $event)"
                        :filters="filters"
                        :filterDisplay="filterDisplay"
                        :filtersStore="filtersStore"
                        @filter-change="$emit('filter-change', $event)"
                        @filter-apply="$emit('filter-apply')"
                        @operator-change="$emit('operator-change', $event)"
                        @matchmode-change="$emit('matchmode-change', $event)"
                        @constraint-add="$emit('constraint-add', $event)"
                        @constraint-remove="$emit('constraint-remove', $event)"
                        @apply-click="$emit('apply-click', $event)"
                        :headerCheckboxIconTemplate="headerCheckboxIconTemplate"
                        :pt="pt"
                    />
                </template>
            </tr>
        </template>
    </thead>
</template>

<script>
import BaseComponent from 'primevue/basecomponent';
import { ObjectUtils } from 'primevue/utils';
import ColumnFilter from './ColumnFilter.vue';
import HeaderCell from './HeaderCell.vue';
import HeaderCheckbox from './HeaderCheckbox.vue';

export default {
    name: 'TableHeader',
    extends: BaseComponent,
    emits: [
        'column-click',
        'column-mousedown',
        'column-dragstart',
        'column-dragover',
        'column-dragleave',
        'column-drop',
        'column-resizestart',
        'checkbox-change',
        'filter-change',
        'filter-apply',
        'operator-change',
        'matchmode-change',
        'constraint-add',
        'constraint-remove',
        'filter-clear',
        'apply-click'
    ],
    props: {
        columnGroup: {
            type: null,
            default: null
        },
        columns: {
            type: null,
            default: null
        },
        rowGroupMode: {
            type: String,
            default: null
        },
        groupRowsBy: {
            type: [Array, String, Function],
            default: null
        },
        resizableColumns: {
            type: Boolean,
            default: false
        },
        allRowsSelected: {
            type: Boolean,
            default: false
        },
        empty: {
            type: Boolean,
            default: false
        },
        sortMode: {
            type: String,
            default: 'single'
        },
        groupRowSortField: {
            type: [String, Function],
            default: null
        },
        sortField: {
            type: [String, Function],
            default: null
        },
        sortOrder: {
            type: Number,
            default: null
        },
        multiSortMeta: {
            type: Array,
            default: null
        },
        filterDisplay: {
            type: String,
            default: null
        },
        filters: {
            type: Object,
            default: null
        },
        filtersStore: {
            type: Object,
            default: null
        },
        reorderableColumns: {
            type: Boolean,
            default: false
        },
        filterInputProps: {
            type: null,
            default: null
        },
        headerCheckboxIconTemplate: {
            type: Function,
            default: null
        }
    },
    methods: {
        columnProp(col, prop) {
            return ObjectUtils.getVNodeProp(col, prop);
        },
        getFilterColumnHeaderClass(column) {
            return [
                'p-filter-column',
                this.columnProp(column, 'filterHeaderClass'),
                this.columnProp(column, 'class'),
                {
                    'p-frozen-column': this.columnProp(column, 'frozen')
                }
            ];
        },
        getFilterColumnHeaderStyle(column) {
            return [this.columnProp(column, 'filterHeaderStyle'), this.columnProp(column, 'style')];
        },
        getHeaderRows() {
            let rows = [];

            let columnGroup = this.columnGroup;

            if (columnGroup.children && columnGroup.children.default) {
                for (let child of columnGroup.children.default()) {
                    if (child.type.name === 'Row') {
                        rows.push(child);
                    } else if (child.children && child.children instanceof Array) {
                        rows = child.children;
                    }
                }

                return rows;
            }
        },
        getHeaderColumns(row) {
            let cols = [];

            if (row.children && row.children.default) {
                row.children.default().forEach((child) => {
                    if (child.children && child.children instanceof Array) cols = [...cols, ...child.children];
                    else if (child.type.name === 'Column') cols.push(child);
                });

                return cols;
            }
        }
    },
    components: {
        DTHeaderCell: HeaderCell,
        DTHeaderCheckbox: HeaderCheckbox,
        DTColumnFilter: ColumnFilter
    }
};
</script>
