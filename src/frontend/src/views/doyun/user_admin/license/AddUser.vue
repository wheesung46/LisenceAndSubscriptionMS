<template>
    <div id="adduser" class="contents">
        <h1>사용자 개별 추가 <code style="color:#3498db; font-size:20px; font-weight:normal"> "{{ license.policy_title }}"</code></h1>
        <br>

        <div id="inner">
            <h4 style="color:gray">사용자 추가 신청서</h4>
            <table class="info-table">
                <tr>
                    <th>사용자 이메일</th>
                    <td><input type="email" style="width:300px" v-model="email"/></td>
                </tr>
                <tr>
                    <th>사용자 비밀번호</th>
                    <td><input type="password" style="width:200px" v-model="password"/></td>
                </tr>
                <tr>
                    <th>사용자 이름</th>
                    <td><input type="text" style="width:200px" v-model="name"/></td>
                </tr>
                <tr>
                    <th>사용자 부서</th>
                    <td><input type="text" style="width:200px" v-model="dept"/></td>
                </tr>                                            
            </table>
            <br>
            <hr style="border-top:1px solid gray">
            <br>
            <br>
            <h4 style="color:gray" align="center">다음 정보로 사용자를 등록합니다.</h4>
            <br>
            <div align="center">
                <button style="width:120px" @click="addUser()">사용자 추가</button>
            </div>
            <br>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    data() {
        return {
            email: '',
            password: '',
            name: '',
            dept: ''
        }
    },
    computed: {
        license: function() {
            return this.$store.state.licenseStore.selectedlicense;
        },
        userInfo: function() {
            return this.$store.state.userinfo.userInfo
        },
    },
    methods: {
        addUser() {
            if(this.email === '') {
                alert('사용자의 이메일을 입력해주세요.')
            } else if (this.password === '') {
                alert('사용자의 비밀번호를 입력해주세요.')
            } else if (this.name === '') {
                alert('사용자의 이름을 입력해주세요.')
            } else if (this.dept === '') {
                alert('사용자의 부서를 입력해주세요.')
            } else if (this.license.current_count === this.license.max_count && this.license.max_count !== 0) {
                alert('현재 사용자가 꽉 차 사용자를 추가할 수 없습니다.')
            } else {
                // <!-- TODO -->
                // <!-- 1. general_user 추가 -->
                // <!-- 2. email로 general_user 검색 후 no 추출 -->
                // <!-- 3. user_no / license_no / user_admin_no 를 사용해 agent 등록 -->
                // <!-- 4. license 테이블 current_count +1 하기 -->
                console.log('this.userInfo.no: ' + this.userInfo.no + '/' + 'this.license.no: ' + this.license.no)
                axios.post('/api/addUser', null, {
                    params: {
                        'email': this.email,
                        'password': this.password,
                        'name': this.name,
                        'dept': this.dept,
                        'adminUserNo': this.userInfo.no,
                        'licenseNo': this.license.no
                    }
                })
                .then(res => { 
                    if(res.data === 'F') {
                        alert('해당 라이선스 내 중복된 이메일이 존해하여, \n입력한 사용자의 정보를 수정했습니다.');
                        window.history.go(-1);
                    } else if (res.data === 'T') {
                        alert('사용자를 추가했습니다.');
                        window.history.go(-1);
                    } else if (res.data === 'M') {
                        alert('사용량이 꽉 차 더 이상 사용자를 추가할 수 없습니다.');
                        window.history.go(-1);
                    }
                });

            }
        }
    }
}
</script>

<style scoped>
    code {
        font-family: 'Jeju Gothic', sans-serif;
    }
    h1 {
        font-size: 4.5rem;
        color:#000000D9;
    }
    .info-table {
        border-top: 2px solid #ccc;
        border-bottom: 2px solid #ccc;
        width: 100%;
        max-width: 100%;
        border-spacing: 0;
        border-collapse: collapse;
        table-layout: fixed;
    }
    .info-table tr {
        border-bottom: 1px solid #ccc;    
        font-size: 15px;
    }
    .info-table th {
        background: #eaeaea;
        font-weight: normal;
        border: none;
        width: 20%;
        text-align: center;
        padding: 15px;
    }
    .info-table td {
        padding: 15px;
        font-weight: normal;
        text-align: left;
    }
    #inner {
        padding: 10px;
        border: 1px solid lightgray;
        border-radius: 5px;
    }
</style>