<template>
  <el-row>
    <el-col :span="4"></el-col>
    <el-col :span="16">
      <el-form :model="form" label-position="right" label-width="180px" :rules="rules">
        <el-form-item label="根证书CN" prop="rootCN">
          <el-input v-model="form.rootCN" placeholder="example.com" clearable></el-input>
        </el-form-item>
        <el-form-item label="服务器CN" prop="serverCN">
          <el-input v-model="form.serverCN" placeholder="server.example.com" clearable></el-input>
        </el-form-item>
        <el-form-item label="服务器DNS列表(可选)">
          <el-input
            v-model="form.dns"
            type="textarea"
            :rows="3"
            placeholder="server.example.com&#10;server2.example.com&#10;server3.example.com"
          ></el-input>
        </el-form-item>
        <el-form-item label="服务器IP列表(可选)">
          <el-input
            v-model="form.ip"
            type="textarea"
            :rows="3"
            placeholder="192.168.1.1&#10;192.168.2.1&#10;192.168.3.1"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit" style="width: 40%;">提&emsp;&emsp;交</el-button>
          <el-button @click="onCopy" style="width: 40%;">复&emsp;&emsp;制&emsp;&emsp;结&emsp;&emsp;果</el-button>
        </el-form-item>
      </el-form>
    </el-col>
    <el-col :span="4"></el-col>
  </el-row>
  <el-divider>复制结果到shell执行</el-divider>
  <el-input v-model="result" type="textarea" :rows="6" readonly></el-input>
</template>

<script>
import { ElMessage } from 'element-plus'
import { copyText } from 'vue3-clipboard'
export default {
  name: 'App',
  data() {
    return {
      form: {
        rootCN: '',
        serverCN: '',
        dns: '',
        ip: ''
      },
      result: '',
      rules: {
        rootCN: [{ required: true, message: '请输入根证书CN', trigger: 'blur' }],
        serverCN: [{ required: true, message: '请输入服务器证书CN', trigger: 'blur' }],
      }
    }
  },
  methods: {
    onSubmit() {
      var ipStr = this.form.ip.trim().split(/[\r\n]/)
        .map(e => e.trim())
        .filter(e => e !== '')
        .map(e => 'IP:' + e).join(',');
      var dnsStr = this.form.dns.trim().split(/[\r\n]/)
        .map(e => e.trim())
        .filter(e => e !== '')
        .map(e => 'DNS:' + e).join(',');
      var ipDnsStr = [ipStr, dnsStr].filter(e => e !== '').join(',');
      this.result = `# 生成根证书
mkdir "${this.form.rootCN}"
cd "${this.form.rootCN}"
openssl genrsa -out ${this.form.rootCN}.key 2048
openssl req -x509 -new -nodes -key  ${this.form.rootCN}.key -subj "/CN=${this.form.rootCN}" -days 5000 -out ${this.form.rootCN}.crt
# 生成服务证书请求
openssl genrsa -out ${this.form.serverCN}.key 2048
openssl req -new -key ${this.form.serverCN}.key -subj "/CN=${this.form.serverCN}" -out ${this.form.serverCN}.csr
# 生成服务证书
openssl x509 -req -in ${this.form.serverCN}.csr -CA ${this.form.rootCN}.crt -CAkey ${this.form.rootCN}.key -CAcreateserial -out ${this.form.serverCN}.crt -days 5000 -extfile <(printf "subjectAltName=${ipDnsStr}")
`
    },
    onCopy() {
      if (this.result === '') {
        ElMessage.warning(`请先填写表单并提交`);
        return
      }
      copyText(this.result, undefined, (error) => {
        if (error) {
          ElMessage.warning(`复制失败！`);
        } else {
          ElMessage.success(`复制成功！`);
        }
      })
    },
  }
}
</script>

<style>
</style>
