<template>
    <div>
        <h1>Contacts</h1>
        <ul class="list-group">
            <li class="list-group-item" v-for="contact in list">
                <strong>{{contact.name}}</strong> {{contact.email}} {{contact.gender}} {{contact.content}}
                <button @click="showContact(contact.id)" class="btn btn-default btn-xs">Edit</button>
                <button @click="deleteContact(contact.id)" class="btn btn-danger btn-xs">Delete</button>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    data: function(){
        return {
            edit:false,
            list:[],
            contact:{
                id:'',
                name:'',
                email:'',
                gender:'',
                content:''
            }
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
    }
}
</script>
