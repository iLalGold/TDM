<template>
    <div class="app-container">


        <tree-table :data="data" :expand-all="expandAll" border>
            <el-table-column label="事件" width="200">
                <template slot-scope="scope">
                    <span style="color:sandybrown">{{ scope.row.event }}</span>
                    <el-tag>{{ scope.row.timeLine+'ms' }}</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="200">
                <template slot-scope="scope">
                    <el-button type="text" @click="message(scope.row)">点击</el-button>
                </template>
            </el-table-column>
        </tree-table>
    </div>
</template>

<script>

    import treeTable from '@/components/common/base/IceTreeTable.vue'
    import treeToArray from '@/utils/evalTree.js'

    export default {
        name: 'CustomTreeTableDemo',
        components: {treeTable},
        data() {
            return {
                expandAll: true,
                data:
                    {
                        id: 1,
                        event: '事件1',
                        timeLine: 100,
                        comment: '无',
                        children: [
                            {
                                id: 2,
                                event: '事件2',
                                timeLine: 10,
                                comment: '无'
                            },
                            {
                                id: 3,
                                event: '事件3',
                                timeLine: 90,
                                comment: '无',
                                children: [
                                    {
                                        id: 4,
                                        event: '事件4',
                                        timeLine: 5,
                                        comment: '无'
                                    },
                                    {
                                        id: 5,
                                        event: '事件5',
                                        timeLine: 10,
                                        comment: '无'
                                    },
                                    {
                                        id: 6,
                                        event: '事件6',
                                        timeLine: 75,
                                        comment: '无',
                                        children: [
                                            {
                                                id: 7,
                                                event: '事件7',
                                                timeLine: 50,
                                                comment: '无',
                                                children: [
                                                    {
                                                        id: 71,
                                                        event: '事件71',
                                                        timeLine: 25,
                                                        comment: 'xx'
                                                    },
                                                    {
                                                        id: 72,
                                                        event: '事件72',
                                                        timeLine: 5,
                                                        comment: 'xx'
                                                    },
                                                    {
                                                        id: 73,
                                                        event: '事件73',
                                                        timeLine: 20,
                                                        comment: 'xx'
                                                    }
                                                ]
                                            },
                                            {
                                                id: 8,
                                                event: '事件8',
                                                timeLine: 25,
                                                comment: '无'
                                            }
                                        ]
                                    }
                                ]
                            }
                        ]
                    }
            }
        },
        methods: {
            message(row) {
                this.$message.info(row.event)
            }
        }
    }
</script>
