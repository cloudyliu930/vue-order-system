<template>
  <div class="login">
    <div class="page-content">
      <div class="wrapper" style="text-align: left;background:#fff;" v-if="token||state==4">
        <mt-navbar v-model="form.typeId">
          <mt-tab-item id="1">{{$t('message.Personage')}}</mt-tab-item>
          <mt-tab-item id="2">{{$t('message.Company')}}</mt-tab-item>
        </mt-navbar>
        <div v-if="form.typeId==2">
          <div v-if="isMobile">
            <mu-text-field :hintText="$t('message.Company_name')" v-model="mobile2.company" @focus="focus" @blur="blur"
                          class="require" />
            <mu-text-field :hintText="$t('message.Mark')" class="require" v-model="mobile2.mark"  />
            <mu-text-field :hintText="$t('message.Company_id_num')"  class="require ism" v-model="mobile2.companyIdNum"
                            />
            <mu-text-field :hintText="$t('message.AdminName')" class="require" v-model="mobile2.personName"
                           />
          </div>
          <div v-if="isEmail">
            <mu-text-field :hintText="$t('message.Company_name')" class="require" v-model="email2.company" @focus="focus" @blur="blur"
                           />
            <mu-text-field :hintText="$t('message.Mark')" class="require" v-model="email2.mark"  />
            <mu-text-field :hintText="$t('message.Company_id_num')" @focus="focus" @blur="blur" class="require" v-model="email2.companyIdNum"
                           />
            <mu-text-field :hintText="$t('message.AdminName')" class="require" v-model="email2.personName" />

            <mu-text-field :hintText="$t('message.Admin_phone_num')" class="require" v-model="email2.phone"
                          />
            <div>
              <mu-text-field :hintText="$t('message.Msg_code')" class="require" v-model="email2.code"  style="width: 60%"/>
              <span class="countBtn"
                    @click="getMsgCode(email2.phone)" v-if="count==60">{{$t('message.click')}}</span>
              <span v-if="count!=60"
                    class="countBtn">
              {{count}}s {{$t('message.resend')}}
            </span>
            </div>
          </div>
        </div>
        <!--*******************************************************************************************-->
        <div v-if="form.typeId==1">
          <div v-if="isMobile">
            <mu-text-field :hintText="$t('message.Name')" class="require" v-model="mobile1.personName" />
          </div>
          <div v-if="isEmail">
            <mu-text-field :hintText="$t('message.Name')" class="require" v-model="email1.personName" />

            <mu-text-field :hintText="$t('message.Phone_number')"  v-model="email1.phone"
                           />
            <div>
              <mu-text-field :hintText="$t('message.Msg_code')"  v-model="email1.code"  style="width: 60%"/>
              <span class="countBtn"
                    @click="getMsgCode(email1.phone)" v-if="count==60">{{$t('message.click')}}</span>
              <span v-if="count!=60"
                    class="countBtn">
              {{count}}s {{$t('message.resend')}}
            </span>
            </div>
          </div>
        </div>
        <div v-if="form.typeId==1">
          <span class="form-group__message" v-if="$v.wechat1.pwd.minLength&&!$v.wechat1.pwd2.sameAs">{{$t('message.Entered_differ')}}.</span>
        </div>
        <div v-if="form.typeId==2">
          <span class="form-group__message" v-if="!$v.wechat2.companyIdNum.minLength||!$v.wechat2.companyIdNum.maxLength||!$v.mobile2.companyIdNum.minLength||!$v.mobile2.companyIdNum.maxLength||!$v.email2.companyIdNum.minLength||!$v.email2.companyIdNum.maxLength">{{$t('message.company_id_error')}}.</span>
          <span class="form-group__message" v-if="!$v.wechat2.pwd.minLength">{{$t('message.password_error')}}.</span>
          <span class="form-group__message" v-if="$v.wechat2.pwd.minLength&&!$v.wechat2.pwd2.sameAs">{{$t('message.Entered_differ')}}.</span>
          <span class="form-group__message" v-if="!$v.mobile2.mark.alpha||!$v.wechat2.mark.alpha||!$v.email2.mark.alpha">{{$t('message.mark_error')}}.</span>

          <span class="form-group__message" v-if="!$v.mobile2.companyIdNum.check_id||!$v.wechat2.companyIdNum.check_id||!$v.email2.companyIdNum.check_id">营业执照号已被使用</span>

          <span class="form-group__message" v-if="!$v.mobile2.company.check_name||!$v.email2.company.check_name">公司名已被注册</span>

          <span class="form-group__message" v-if="!$v.email2.phone.minLength||!$v.email2.phone.maxLength||!$v.wechat2.phone.minLength||!$v.wechat2.phone.maxLength">{{$t('message.phone_error')}}.</span>


        </div>

        <button class="Button--primary Button--blue" @click="handleSubmit">{{$t('message.Submit')}}</button>
      </div>
      <div style="padding: 20px" v-if="isEmail&&!token">
        <h2>{{$t('message.Links_expire')}}
          <router-link to="register">{{$t('message.Resend_message')}}</router-link>
        </h2>
      </div>
    </div>

  </div>
</template>
<script>
  import { GetQueryString } from '@/utils'
  import Cookies from 'js-cookie'
  import { validationMixin } from 'vuelidate'
  import { required, minLength, maxLength, sameAs, email,requiredUnless ,alpha}from 'vuelidate/lib/validators'
  export default {
    name: 'login',
    data () {
      return {
        form: {
          typeId: '1', // 2 企业 1 个人
        },
        mobile1:{
          personName: '',
        },
        mobile2:{
          company: '',
          mark: '',
          companyIdNum: '', // 营业执照号
          userName: '',
        },
        email1:{
          personName: '',
          phone: '',
          code: '',
        },
        email2:{
          company: '',
          mark: '',
          companyIdNum: '', // 营业执照号
          personName: '',
          phone: '',
          code: '',
        },
        wechat1:{
          personName: '',
          phone: '',
          code: '',
          pwd: '',
          pwd2: '',
        },
        wechat2:{
          company: '',
          mark: '',
          companyIdNum: '', // 营业执照号
          personName: '',
          phone: '',
          code: '',
          pwd: '',
          pwd2: '',
        },
        state: '', // state=4 位信息不完善
        editData: {},
        count: 60,
        token: true,
        isEmail: false,
        isMobile: false, // 是否手机号登录
        isWechat: false, // 是否微信进入
        cck:false,
      }
    },
    validations: {
      mobile1: {
        personName: {
          required,
        },
      },
      mobile2: {
        company: {
          required,
          check_name(value) {
            if(!this.cck){
              return true
            }
            if(value.length<3){
              return true
            }
            return new Promise((resolve, reject) => {
              this.$api.CHECK_COMPANY_NAME({companyName: value}).then(res => {
                if (res.code === 0) {
                  if (res.state === 0) {
                    resolve(true)
                  } else if (res.state === 1) { // 不可用
                    resolve(false)
                  }
                }else {
                  MessageBox.alert(JSON.stringify(res))
                }
              })
            })
          }
        },
        mark:{
          required,
          alpha
        },
        companyIdNum:{
          required,
          minLength: minLength(15),
          maxLength: maxLength(18),
          check_id(value) {
            if(value.length<15){
              return true
            }
            return new Promise((resolve, reject) => {
              this.$api.CHECK_BUSINESS_LICENSE({businessLicenseNumber: value}).then(res => {
                if (res.code === 0) {
                  if (res.state === 0) {
                    resolve(true)
                  } else if (res.state === 1) { // 不可用
                    resolve(false)
                  }
                }else {
                  MessageBox.alert(JSON.stringify(res))
                }
              })
            })
          }
        },
        personName:{
          required,
        }
      },
      email1:{
        personName: {
          required,
        },
      },
      email2:{
        company: {
          required,
          check_name(value) {
            if(!this.cck){
              return true
            }
            if(value.length<3){
              return true
            }
            return new Promise((resolve, reject) => {
              this.$api.CHECK_COMPANY_NAME({companyName: value}).then(res => {
                if (res.code === 0) {
                  if (res.state === 0) {
                    resolve(true)
                  } else if (res.state === 1) { // 不可用
                    resolve(false)
                  }
                }else {
                  MessageBox.alert(JSON.stringify(res))
                }
              })
            })
          }
        },
        mark:{
          required,
          alpha
        },
        companyIdNum:{
          required,
          minLength: minLength(15),
          maxLength: maxLength(18),
          check_id(value) {
            if(value.length<15){
              return true
            }
            return new Promise((resolve, reject) => {
              this.$api.CHECK_BUSINESS_LICENSE({businessLicenseNumber: value}).then(res => {
                if (res.code === 0) {
                  if (res.state === 0) {
                    resolve(true)
                  } else if (res.state === 1) { // 不可用
                    resolve(false)
                  }
                }else {
                  MessageBox.alert(JSON.stringify(res))
                }
              })
            })
          }
        },
        personName:{
          required,
        },
        phone:{
          required,
          minLength:minLength(11),
          maxLength:maxLength(11),
        },
        code:{
          required,
        },

      },
      wechat1:{
        personName: {
          required,
        },
        pwd: {
          required,
          minLength: minLength(6),
        },
        pwd2: {
          sameAs: sameAs('pwd'),
        }
      },
      wechat2:{
        company: {
          required,
          check_name(value) {
            if(!this.cck){
              return true
            }
            if(value.length<3){
              return true
            }
            return new Promise((resolve, reject) => {
              this.$api.CHECK_COMPANY_NAME({companyName: value}).then(res => {
                if (res.code === 0) {
                  if (res.state === 0) {
                    resolve(true)
                  } else if (res.state === 1) { // 不可用
                    resolve(false)
                  }
                }else {
                  MessageBox.alert(JSON.stringify(res))
                }
              })
            })
          }
        },
        mark:{
          required,
          alpha
        },
        companyIdNum:{
          required,
          minLength: minLength(15),
          maxLength: maxLength(18),
          check_id(value) {
            if(value.length<15){
              return true
            }
            return new Promise((resolve, reject) => {
              this.$api.CHECK_BUSINESS_LICENSE({businessLicenseNumber: value}).then(res => {
                if (res.code === 0) {
                  if (res.state === 0) {
                    resolve(true)
                  } else if (res.state === 1) { // 不可用
                    resolve(false)
                  }
                }else {
                  MessageBox.alert(JSON.stringify(res))
                }
              })
            })
          }
        },
        personName:{
          required,
        },
        phone:{
          required,
          minLength:minLength(11),
          maxLength:maxLength(11),
        },
        code:{
          required,
        },
        pwd: {
          required,
          minLength: minLength(6),
        },
        pwd2: {
          sameAs: sameAs('pwd'),
        }
      }
    },
    beforeCreate () {
      // 获取地址栏地址
      let uri = window.location.href
      uri=decodeURIComponent(uri)
      window.location.href = uri
      if (uri.indexOf('&amp;') !== -1) {
        uri = window.location.href.replace(/&amp;/g, '&')
        window.location.href = uri
      }
      if (uri.indexOf('&;') !== -1) {
        uri = window.location.href.replace(/&;/g, '&')
        window.location.href = uri
      }
    },
    methods: {
      back(){
        this.$router.back()
      },
      focus() {
        this.cck = false
      },
      blur() {
        this.cck = true
      },
      handleSubmit () {
        if(this.isMobile){ // 移动端
          if(this.form.typeId==1){ // 个人
            if (!this.$v.mobile1.$invalid) {
              let data={
                account_type: this.form.typeId,
                personName: this.mobile1.personName,
                table_name: GetQueryString('table_name') ||(Cookies.getJSON('table_id') && Cookies.getJSON('table_id').table_name) || '',
                table_id: GetQueryString('table_id') ||(Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_id )|| '',
                email: GetQueryString('e') || '',
                source:( GetQueryString('source')=='wechat'?'telephone': GetQueryString('source') )|| 'telephone'
              }
              this._postData(data)
            }else{
              alert(this.$t('message.check_fields'))
            }
          }else{// 企业
            if (!this.$v.mobile2.$invalid) {
              let data={
                account_type: this.form.typeId,
                company_name: this.mobile2.company,
                comp_eng_short_name: this.mobile2.mark,
                business_license_number: this.mobile2.companyIdNum,
                personName: this.mobile2.personName,
                table_name: GetQueryString('table_name') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_name )|| '',
                table_id: GetQueryString('table_id') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_id )|| '',
                email: GetQueryString('e') || '',
                source: GetQueryString('source') || 'telephone'
              }
              this._postData(data)
            }else{
              alert(this.$t('message.check_fields'))
            }
          }

        }else if(this.isEmail){ // 邮箱
            if(this.form.typeId==1){ // 个人
              if(!this.$v.email1.$invalid){
                if(this.email1.phone.length>0){
                  if(this.email1.code.length==0){
                    alert('请填写验证码')
                    return
                  }
                }
                let data={
                  account_type: this.form.typeId,
                  personName: this.email1.personName,
                  table_name: GetQueryString('table_name') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_name )|| '',
                  table_id: GetQueryString('table_id') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_id )|| '',
                  email: GetQueryString('e') || '',
                  source: GetQueryString('source') || '',
                  telephone: this.email1.phone,
                  code: this.email1.code,
                }
                this._postData(data)
              }else{
                alert(this.$t('message.check_fields'))
              }
            }else{
              if(!this.$v.email2.$invalid){
                let data = {
                  account_type: this.form.typeId,
                  company_name: this.email2.company,
                  comp_eng_short_name: this.email2.mark,
                  business_license_number: this.email2.companyIdNum,
                  personName: this.email2.personName,
                  telephone: this.email2.phone,
                  table_name: GetQueryString('table_name') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_name )|| '',
                  table_id: GetQueryString('table_id') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_id) || '',
                  email: GetQueryString('e') || '',
                  code: this.email2.code,
                  source: GetQueryString('source') || ''
                }
                this._postData(data)
              }else{
                alert(this.$t('message.check_fields'))
              }

            }
        }else if(this.isWechat){
            if(this.form.typeId==1){ // 个人
              if(!this.$v.wechat1.$invalid){
                if(this.wechat1.phone.length>0){
                  if(this.wechat1.code.length==0){
                    alert("请填写验证码")
                    return
                  }
                }
                let data={
                  account_type: this.form.typeId,
                  personName: this.wechat1.personName,
                  table_name: GetQueryString('table_name') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_name )|| '',
                  table_id: GetQueryString('table_id') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_id) || '',
                  email: GetQueryString('e') || '',
                  source: GetQueryString('source') || '',
                  telephone: this.wechat1.phone,
                  code: this.wechat1.code,
                  password:this.wechat1.pwd,
                }
                this._postData(data)
              }else{
                alert(this.$t('message.check_fields'))
              }

            }else{ // 企业
              if(!this.$v.wechat2.$invalid){
                let data = {
                  account_type: this.form.typeId,
                  company_name: this.wechat2.company,
                  comp_eng_short_name: this.wechat2.mark,
                  business_license_number: this.wechat2.companyIdNum,
                  personName: this.wechat2.personName,
                  telephone: this.wechat2.phone,
                  table_name: GetQueryString('table_name') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_name) || '',
                  table_id: GetQueryString('table_id') || (Cookies.getJSON('table_id') &&Cookies.getJSON('table_id').table_id )|| '',
                  email: GetQueryString('e') || '',
                  code: this.wechat2.code,
                  source: GetQueryString('source') || '',
                  password:this.wechat2.pwd,
                }
                this._postData(data)
              }else{
                alert(this.$t('message.check_fields'))
              }
            }
        }
      },
      _postData(data){
        this.$api.POST_USER_INFO(data).then(res => {
          this.loading = false
          if (res.code === 0) {
            if (res.state !== '1') { // 验证码不通过
              alert(res.info)
              return
            }
            let url = window.location.href
            if (window.location.search === '') {
              url = window.location.href
            } else {
              url = url.split(window.location.search)[0]
            }
            if (this.isWechat) {
              url = url.replace('type', 'index')
              window.location.href = url
            } else {
              url = url.replace('type', 'login')
              window.location.href = url
            }
          } else {
            alert(JSON.stringify(res))
          }
        }).catch(err => {
          this.loading = false
          console.error('哎哟~！', err)
        })
      },
      getMsgCode (val) {
        if (val.length != 11)return
        let data = {
          telephone: val,
          type: 0
        }
        this.count--
        let s = setInterval(() => {
          this.count--
          if (this.count == 0) {
            this.count = 60
            clearInterval(s)
          }
        }, 1000)

        this.$api.GET_MSG_CODE(data).then(res => {
          if (res.code === 0) {
            if (res.state === 1) {
              this.$toast('短信已发出，请查收')
            } else {
              alert(JSON.stringify(res))
            }
          } else {
            alert(JSON.stringify(res))
          }
        })
      },
      checkToken () {
        let data = {
          token: GetQueryString('token') || '',
          e: GetQueryString('e') || '',
          p: GetQueryString('p') || ''
        }
        // 如果有source=email 就是从邮件来的 就必须验证token

        if (GetQueryString('source') && GetQueryString('source') == 'email') {
          this.isEmail = true
          this.isWechat = false
          this.isMobile = false
          this.$api.CHECK_TOKEN(data).then(res => {
            if (res.code === 0) {
              if (res.state === '1') {
                this.token = true
                // console.log('token有效')
              } else {
                // console.log('token无效')
                this.token = false
              }
            } else {
              alert(JSON.stringify(res))
            }
          })
        } else if (GetQueryString('source') && GetQueryString('source') == 'wechat') {
//          this.isWechat = true
          this.isWechat = false
          this.isEmail = false
//          this.isMobile = false
          this.isMobile = true
          this.$api.CHECK_ACCOUNT().then(res => { // 判断注册信息是否完善
            if (res.code === 0) {
              if (res.state == 4) { // 不完善
                this.state = 4
//                   this.checkMobile() // 信息不全的时候还得判断是邮箱的注册的信息不全还是手机注册的信息不全
              } else {
                this.$router.push('/index');
              }
            } else {
              alert(JSON.stringify(res))
            }
          })
        } else if (GetQueryString('state') && GetQueryString('state') == '4') {
          this.checkMobile()
        } else {
          this.isMobile = true
          this.isEmail = false
          this.isWechat = false
        }
      },
      checkMobile () { // 编辑分离出去后这个方法似乎不需要了
        this.$api.CHECK_BIND_MOBILE().then(res => {
          if (res.code === 0) {
            if (res.exist === '1') { // 是手机号登录(注册)
              this.isMobile = true
            } else {
              this.isMobile = false
              if (res.exist === '2') { //2 微信 3 邮箱
//                this.isWechat = true
                this.isMobile=true
                this.isWechat = false
              } else {
                this.isWechat = false
              }
              if (res.exist === '3') {
                this.isEmail = true
              } else {
                this.isEmail = false
              }
            }
          } else {
            alert(JSON.stringify(res))
          }
        })
      },
    },
    created () {

    },
    activated(){
      this.checkToken()
    },
    mounted(){
    }
  }
</script>
<style scoped lang="less">
  .Button--primary.Button--blue {
    border: none;
    outline: none;
    width: 100%;
    color: #fff;
    padding: 10px;
    background: -webkit-gradient(linear, left top, right top, from(#15b982), to(#01d18b));
    background: linear-gradient(90deg, #15b982, #01d18b);
    -webkit-box-shadow: 0 3px 6px rgba(0, 0, 0, .16);
    box-shadow: 0 3px 6px rgba(0, 0, 0, .16);
  }

  .help {
    font-size: 10px;
    color: rgba(206, 129, 17, 0.64);
    padding-left: 10px;
  }

  .help.is-danger {
    font-size: 10px;
    color: #f00;
    text-align: left;
    padding-left: 10px;
  }

</style>
