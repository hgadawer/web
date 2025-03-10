<template>
    <div :style="{ padding: '20px 20px 12px 20px' }">
        <div style="display: flex;justify-content: space-between;margin-bottom: 20px;">
            <a-space>
                <a-input v-model:value="keyWord" placeholder="产品名称" style="width: 280px; margin-right: 10px;">
                    <template #suffix>
                        <search-outlined style="color: rgba(0, 0, 0, 0.45)" @click="productList(0)" />
                    </template>
                </a-input>
                <a-button :type="buttonType.bt1" @click="productList(0)">全部产品</a-button>
                <a-button :type="buttonType.bt2" @click="productList(1)">上架的产品</a-button>
                <a-button :type="buttonType.bt3" @click="productList(2)">下架的产品</a-button>
                <a-button type="primary" @click="onDelete" :disabled="disabled" danger>删除</a-button>
            </a-space>
            <div>
                <a-space size="middle">
                    <a-button type="primary" @click="onCreate">新建</a-button>
                <a-button type="primary" @click="onExport" ghost>
                    <template #icon>
                        <ExportOutlined />
                    </template>导出</a-button>
                </a-space>
            </div>
        </div>
        <a-table rowKey="id" :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
            :columns="columns" :data-source="data.productList"
            :pagination="{ current: pagination.current, pageSize: pagination.pageSize, total: pagination.total, onChange: onPagination }"
            :scroll="{ y: '59vh' }" class="ant-table-striped"
            :row-class-name="(_record, index) => (index % 2 === 1 ? 'table-striped' : null)" bordered>
            <template #bodyCell="{ column, text, record }">
                <template v-if="column.dataIndex === 'name'">
                    <a @click="onEdit(record)">{{ text }}</a>
                </template>
                <template v-if="column.dataIndex === 'status'">
                    <Spot v-if="text == 1" type="success" title="上架" />
                    <Spot v-if="text == 2" type="danger" title="下架" />
                </template>
                <template v-if="column.dataIndex === 'type'">
                    <span v-if="text == 1">默认</span>
                </template>
                <template v-if="column.dataIndex === 'price'">
                    <span style="color: #ff991f">{{ text }}</span>
                </template>
            </template>
        </a-table>
        <!-- 新建、编辑产品 -->
        <a-modal v-model:visible="visible" :title="title" @ok="onSave" @cancel="onCancel" cancelText="取消" okText="保存"
            width="800px" :centered="true">
            <div style="height: 55vh;overflow-y: scroll;padding: 0 15px;">
                <a-form ref="productFormRef" :model="product" layout="vertical" name="product" :rules="rules">
                    <a-row :gutter="16">
                        <a-col :span="12">
                            <a-form-item label="产品名称" name="name">
                                <a-input v-model:value="product.name" />
                            </a-form-item>
                        </a-col>
                        <a-col :span="12">
                            <a-form-item label="产品类型" name="type">
                                <a-select v-model:value="product.type" placeholder="请选择">
                                    <a-select-option :value="1">默认</a-select-option>
                                </a-select>
                            </a-form-item>
                        </a-col>
                    </a-row>
                    <a-row :gutter="16">
                        <a-col :span="12">
                            <a-form-item label="产品单位" name="unit">
                                <a-select v-model:value="product.unit" placeholder="请选择">
                                    <a-select-option value="个">个</a-select-option>
                                    <a-select-option value="只">只</a-select-option>
                                    <a-select-option value="块">块</a-select-option>
                                    <a-select-option value="瓶">瓶</a-select-option>
                                    <a-select-option value="盒">盒</a-select-option>
                                    <a-select-option value="台">台</a-select-option>
                                    <a-select-option value="箱">箱</a-select-option>
                                    <a-select-option value="吨">吨</a-select-option>
                                </a-select>
                            </a-form-item>
                        </a-col>
                        <a-col :span="12">
                            <a-form-item label="产品编码" name="code">
                                <a-input v-model:value="product.code" />
                            </a-form-item>
                        </a-col>
                    </a-row>
                    <a-row :gutter="16">
                        <a-col :span="12">
                            <a-form-item label="价格" name="price">
                                <a-input-number v-model:value="product.price" style="width: 100%" />
                            </a-form-item>
                        </a-col>
                        <a-col :span="12">
                            <a-form-item label="是否上下架" name="status">
                                <a-select v-model:value="product.status" placeholder="请选择">
                                    <a-select-option :value="1">
                                        <Spot type="success" title="上架" />
                                    </a-select-option>
                                    <a-select-option :value="2">
                                        <Spot type="danger" title="下架" />
                                    </a-select-option>
                                </a-select>
                            </a-form-item>
                        </a-col>
                    </a-row>
                    <a-row :gutter="16">
                        <a-col :span="24">
                            <a-form-item label="产品描述" name="description">
                                <a-textarea v-model:value="product.description" :rows="4" />
                            </a-form-item>
                        </a-col>
                    </a-row>
                </a-form>
            </div>
        </a-modal>
    </div>
</template>

<script setup>
import { ref, reactive, onMounted, createVNode } from 'vue';
import { SearchOutlined, ExclamationCircleOutlined, ExportOutlined } from '@ant-design/icons-vue';
import moment from 'moment'
import { createProduct, updateProduct, queryProductList, deleteProduct, queryProductInfo, productExport } from '../api/product';
import Spot from '../components/Spot.vue';
import { message, Modal } from 'ant-design-vue';

// 表格字段
const columns = [{
    title: '产品名称',
    dataIndex: 'name',
    width: 100,
    fixed: 'left',
    ellipsis: true,
}, {
    title: '是否上下架',
    dataIndex: 'status',
    width: 120,
}, {
    title: '产品类型',
    dataIndex: 'type',
    width: 100,
}, {
    title: '产品单位',
    dataIndex: 'unit',
    width: 100,
}, {
    title: '产品编码',
    dataIndex: 'code',
    width: 150,
}, {
    title: '价格',
    dataIndex: 'price',
    width: 150,
}, {
    title: '产品描述',
    dataIndex: 'description',
    width: 240,
    ellipsis: true,
}, {
    title: '创建时间',
    dataIndex: 'created',
    width: 185,
    customRender: ({text}) => {
        console.log(text)
        return text ? moment(text).format('YYYY-MM-DD HH:mm:ss') : '';
    }
}, {
    title: '更新时间',
    dataIndex: 'updated',
    width: 185,
    customRender: ({text}) => {
        return text ? moment(text).format('YYYY-MM-DD HH:mm:ss') : '';
    }
}];

// 表单校验
const rules = {
    name: [{ required: true, message: '请输入产品名称', trigger: 'blur' }],
    type: [{ required: true, message: '请选择产品类型' }],
    code: [{ pattern: /^\d+$/, message: '产品编码格式不正确', trigger: 'blur' }],
    price: [{ required: true, message: '请输入产品价格' }],
    status: [{ required: true, message: '请选择是否上下架' }]
};

// 产品属性
let product = reactive({
    id: undefined,
    name: undefined,
    type: undefined,
    unit: undefined,
    code: undefined,
    price: undefined,
    status: undefined,
    description: undefined,
});

// 产品列表
const data = reactive({
    productList: [],
    selectedIds: []
})

// 表格分页
let pagination = reactive({
    current: 1,
    pageSize: 10,
    total: undefined,
})

const title = ref('');
const visible = ref(false);
const disabled = ref(true)
const operation = ref(0);
const productFormRef = ref();
const keyWord = ref('')

// 按钮状态
const buttonType = reactive({
    bt1: 'primary',
    bt2: 'default',
    bt3: 'default',
})
const setButtonType = (status) => {
    if (status == 0) {
        buttonType.bt1 = 'primary'
        buttonType.bt2 = 'default'
        buttonType.bt3 = 'default'
    }
    if (status == 1) {
        buttonType.bt1 = 'default'
        buttonType.bt2 = 'primary'
        buttonType.bt3 = 'default'
    }
    if (status == 2) {
        buttonType.bt1 = 'default'
        buttonType.bt2 = 'default'
        buttonType.bt3 = 'primary'
    }
}

// 初始化数据
onMounted(() => { productList() })

// 表格记录多选
const onSelectChange = selectedRowKeys => {
    data.selectedIds = selectedRowKeys
    if (data.selectedIds.length !== 0) {
        disabled.value = false
    } else {
        disabled.value = true
    }
};

// 点击新建产品
const onCreate = () => {
    title.value = '新建产品'
    operation.value = 1
    visible.value = true
}

// 点击产品名称
const onEdit = (row) => {
    title.value = '编辑产品'
    operation.value = 2
    let param = { id: row.id }
    queryProductInfo(param).then((res) => {
        if (res.data.code == 200) {
            let p = res.data.info
            product.id = p.id
            product.name = p.name
            product.type = p.type
            product.unit = p.unit
            product.code = p.code
            product.price = p.price
            product.status = p.status
            product.description = p.description
        }
    })
    visible.value = true
}

// 点击保存产品
const onSave = () => {
    productFormRef.value.validateFields().then(() => {
        if (operation.value == 1) {
            createProduct(product).then((res) => {
                if (res.data.code == 200) {
                    message.success('保存成功')
                    productFormRef.value.resetFields()
                    visible.value = false;
                    productList()
                }
                if (res.data.code == 500) {
                    message.error('产品名称已经存在')
                }
            })
        }
        if (operation.value == 2) {
            updateProduct(product).then((res) => {
                if (res.data.code == 200) {
                    productFormRef.value.resetFields()
                    visible.value = false;
                    message.success('保存成功')
                    productList()
                }
            })
        }
    });
};

// 点击删除产品
const onDelete = () => {
    Modal.confirm({
        title: '确定删除选中的' + data.selectedIds.length + '项吗?',
        icon: createVNode(ExclamationCircleOutlined),
        centered: true,
        cancelText: '取消',
        okText: '确定',
        onOk() {
            deleteProduct({ ids: data.selectedIds }).then((res) => {
                if (res.data.code == 200) {
                    productList()
                    disabled.value = true
                    message.success('删除成功')
                }
            })
        },
        onCancel() {
            console.log('Cancel');
        },
    });
}

// 分页查询产品列表
const onPagination = (page) => {
    pagination.current = page
    productList()
}

// 获取产品列表
const productList = (status) => {
    setButtonType(status)
    let param = {
        name: keyWord.value,
        status: status,
        pageNum: pagination.current,
        pageSize: pagination.pageSize,
    }
    queryProductList(param).then((res) => {
        if (res.data.code == 200) {
            pagination.total = res.data.info.total
            data.productList = res.data.info.list
        }
    })
}

// 导出表格
const onExport = () => {
    productExport().then((res) => {
        // 检查后端返回状态码
        if (res.data.code !== 200) {
            message.error(res.data.msg || '导出错误！');
            return;
        }

        // 提取Base64数据和文件名
        const { data: base64Data, fileName } = res.data.info;

        // 将Base64转换为二进制
        const byteCharacters = atob(base64Data);
        const byteNumbers = new Array(byteCharacters.length);
        for (let i = 0; i < byteCharacters.length; i++) {
            byteNumbers[i] = byteCharacters.charCodeAt(i);
        }
        const byteArray = new Uint8Array(byteNumbers);

        // 创建Blob对象，使用更精确的xlsx MIME类型
        const blob = new Blob([byteArray], {
            type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
        });

        // 创建下载链接
        const a = document.createElement('a');
        a.setAttribute("download", fileName || "产品信息.xlsx"); // 使用后端返回的文件名
        a.href = window.URL.createObjectURL(blob);
        a.click();
        window.URL.revokeObjectURL(a.href);

        message.success('导出成功');
    }).catch((error) => {
        message.error('导出失败：' + error.message);
    });
};

// 点击取消按钮
const onCancel = () => {
    productFormRef.value.resetFields()
    visible.value = false
}
</script>

<style scoped>
.ant-table-striped :deep(.table-striped) td {
    background-color: #fafafa;
}
</style>