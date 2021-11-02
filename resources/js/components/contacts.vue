<template>
    <div v-if="user_id !== ''">
        <h1>Contacts</h1>

        <div class="form-group">
            <input type="text" id="myInput" onkeyup="myFunction()" placeholder="Search">
        </div>

        <table class="table table-dark table-striped" id="myTable">
            <thead>
            <tr>
                <th>NAME</th>
                <th>EMAIL</th>
                <th>GENDER</th>
                <th>CONTENT</th>
            </tr>
            </thead>
            <tbody v-for="contact in list">
            <tr>
                <td>{{contact.name}}</td>
                <td>{{contact.email}}</td>
                <td>{{contact.gender}}</td>
                <td>{{contact.content}}</td>
            </tr>
            </tbody>
        </table>
    </div>

    <div v-else>
        <h1>Add Contact</h1>
        <div class="alert alert-success" v-show="success">
            Contact stored successfully
        </div>
        <form ref="form" action="#" @submit.prevent="edit ? updateContact(contact.id) : createContact()">
            <div class="form-group">
                <label>Name</label>
                <input v-model="contact.name" type="text" name="name" class="form-control">
            </div>
            <div class="form-group">
                <label>Email</label>
                <input v-model="contact.email" type="text" name="email" class="form-control">
            </div>
            <div class="form-group">
                <label>Gender</label>
                <select v-model="contact.gender" name="gender" class="form-control">
                    <option v-for="gender in genders">{{gender}}</option>
                </select>
            </div>
            <div class="form-group">
                <label>Content</label>
                <input v-model="contact.content" type="text" name="content" class="form-control">
            </div>
            <div class="form-group">
                <button v-show="!edit" type="submit" class="btn btn-primary">New Contact</button>
                <button v-show="edit" type="submit" class="btn btn-primary">Update Contact</button>
            </div>
        </form>
    </div>

</template>

<script>
import emailjs from 'emailjs-com';

export default {
    props: ['app'],
    data: function(){
        return {
            user_id: this.$userId,
            edit:false,
            list:[],
            contact:{
                id:'',
                name:'',
                email:'',
                gender:'',
                content:''
            },
            success: false,
            genders:['Male', 'Female']
        }
    },
    mounted: function(){
        console.log('Contacts Component Loaded...');
        this.fetchContactList();
    },
    methods: {
        fetchContactList:function(){
            console.log('Fetching contacts...');
            axios.get('api/contacts')
                .then((response) => {
                    console.log(response.data);
                    this.list = response.data;
                }).catch((error) => {
                console.log(error);
            });
        },
        createContact: function(){
            console.log('Creating contact...');
            let self = this;
            let params =Object.assign({}, self.contact);
            axios.post('api/contact/store', params)
                .then(function(){
                    self.sendEmail();
                    self.contact.name = '';
                    self.contact.email = '';
                    self.contact.gender = '';
                    self.contact.content = '';
                    self.edit = false;
                    self.success = true;

                    self.fetchContactList();
                })
                .catch(function(error){
                    console.log(error);
                });
        },
        sendEmail() {
            emailjs.sendForm('YOUR_SERVICE_ID', 'YOUR_TEMPLATE_ID', this.$refs.form, 'YOUR_USER_ID',
                {
                    name: this.name,
                    email: this.email,
                    gender: this.gender,
                    content: this.content
                })
                .then((result) => {
                    console.log('SUCCESS!', result.text);
                }, (error) => {
                    console.log('FAILED...', error.text);
                });
        },
        showContact: function(id){
            let self = this;
            axios.get('api/contact/'+id)
                .then(function(response){
                    self.contact.id = response.data.id;
                    self.contact.name = response.data.name;
                    self.contact.email = response.data.email;
                    self.contact.gender = response.data.gender;
                    self.contact.content = response.data.content;
                })
            self.edit = true;
        },
        updateContact: function(id){
            console.log('Updating contact '+id+'...');
            let self = this;
            let params =Object.assign({}, self.contact);
            axios.patch('api/contact/'+id, params)
                .then(function(){
                    self.contact.name = '';
                    self.contact.email = '';
                    self.contact.gender = '';
                    self.contact.content = '';
                    self.edit = false;
                    self.fetchContactList();
                })
                .catch(function(error){
                    console.log(error);
                });
        },
        deleteContact: function(id){
            axios.delete('api/contact/'+id)
                .then(function(response){
                    self.fetchContactList();
                })
                .catch(function(error){
                    console.log(error);
                });
        }
    },
    created(){
        console.log(this.$userId);
        console.log(this.$serviceId);
        console.log(this.$templateId);
        console.log(this.$uId);
    }
}
</script>
