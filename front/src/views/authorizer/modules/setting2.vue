<template>
  <div>
    <a-card :bordered="false" title="小程序业务域名">
      <a-button slot="extra" type="primary" @click="changeDomain">
        修改
      </a-button>
      <a-descriptions :column=1 bordered>
        <a-descriptions-item label="业务域名">
          {{ domain['BizDomain'] | join(';') }}
        </a-descriptions-item>
      </a-descriptions>
    </a-card>
    <a-modal
      v-model="changeDomainModal"
      title="小程序业务域名"
      width="800px"
      @cancel="domainCancel"
      @ok="domainOk">
      <a-form :form="form" v-bind="formLayout">
        <!-- 检查是否有 id 并且大于0，大于0是修改。其他是新增，新增不显示主键ID -->
        <a-form-item label="合法业务域名">
          <a-input v-decorator="['webviewdomain']" placeholder="以 https:// 开头。可填写50个域名，域名间请用 ; 分割"/>
        </a-form-item>
      </a-form>
    </a-modal>

  </div>
</template>

<script>
import {eventBus} from '@/main'
import {setDomainBiz} from "@/api/miniprogram";

// 表单字段
const fields = ['webviewdomain']

export default {
  name: 'Setting',
  data() {
    this.formLayout = {
      labelCol: {
        xs: {span: 24},
        sm: {span: 7}
      },
      wrapperCol: {
        xs: {span: 24},
        sm: {span: 13}
      }
    }
    return {
      id: null,
      profile: null,
      changeDomainModal: false,
      form: this.$form.createForm(this)
    }
  },
  computed: {
    domain() {
      if (this.profile) {
        return this.profile['authorizer_info']['MiniProgramInfo']['network']
      } else {
        return []
      }
    }
  },
  methods: {
    changeDomain() {
      this.changeDomainModal = true
      let data = {
        webviewdomain: this.domain['BizDomain'] ? this.domain['BizDomain'].join(';') : ''
      }
      this.form.setFieldsValue(data)

    },
    domainOk() {
      this.form.validateFields((errors, values) => {
        if (!errors) {
          for (let key in values) {
            values[key] = values[key] ? values[key].split(';') : [];
          }
          setDomainBiz({id: this.id, ...values}).then(res => {
            this.visible = false
            this.form.resetFields()
            this.$message.success(res['msg'])
            setTimeout(() => {
              location.reload()
            }, 1500)
          }).catch(e => {
            console.log(e)
          }).finally(() => {
          })
        }
      })
    },
    domainCancel() {

    }
  },
  created() {
    const {id} = this.$route.query
    this.id = id
    // 防止表单未注册
    fields.forEach(v => this.form.getFieldDecorator(v, {}))
  },
  mounted() {
    eventBus.$on('profile-update', (value) => {
      this.profile = JSON.parse(value)
    });
  }
}
</script>

<style lang="less" scoped>

</style>
