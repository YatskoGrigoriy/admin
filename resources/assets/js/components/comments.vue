<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-8">

        <span v-if="currentUser!==undefined">
        <h4>Add Comment</h4>

        <form action="" @submit.prevent="edit ? editComment(comment.id) : createComment()">
          <div class="input-group">
            <textarea name="body" v-model="comment.body" ref="textarea"  class="form-control"></textarea>
          </div>
          
            <div class="input-group">
              <button type="submit" class="btn btn-primary" v-show="!edit">Add Comment</button>
              <button type="submit" class="btn btn-primary" v-show="edit">Edit Comment</button>
            </div>
        </form>
        </span>
        <h4>Comments</h4>
          <ul class="list-group">
            <li class="list-group-item" v-for="comment in comments">
              {{comment.body}}
              {{comment.creator_id}}
              <span v-if="(currentUser!==undefined && currentUser===comment.creator_id)">
              <a class="btn btn-default" v-on:click=" showComment(comment.id)">Edit</a>
              <a class="btn btn-danger" v-on:click=" deleteComment(comment.id)">Delete</a>
              </span>
            </li>

          </ul>
          <ul v-if="errors && errors.length">
            <li v-for="error of errors">
              {{error.message}}
            </li>
          </ul>
      </div>
    </div>
  </div>
</template>

<script>
  
  export default {

    props: ['currentId','currentUser'],
    
    data: () => ({
      edit: false,
      comments:[],
      comment: {
          title:'',
          body: '',
          id: '',
      
        },
      errors: []
    }),


  created: function(){
    this.fetchComments();
  },
  ready: function(){
    this.fetchComments();
  },
  
  // mounted: function(){
  //  this.fetchComments();
  // },

  
  methods: {
    fetchComments: function(){
      this.$http.get("../api/post/"+this.currentId+"/comments")
        .then(response => {
          this.comments = response.data;
          // this.user_id = this.currentUser;
      })
      .catch(error => {
        this.errors.push(error)
      });
    },

    createComment: function(){
      this.$http.post("../api/post/"+this.currentId+"/comment", this.comment)
        .then( (response) => {
          this.comment.body= '';
          this.fetchComments();
      }).catch( error => {
         this.comment.body = '';
         this.fetchComments();
        });
    },

    editComment: function(comment_id){
      this.$http.patch("../api/post/"+this.currentId+"/comment/"+comment_id, this.comment)
        .then( response => {
          this.comment.body= '';
          this.comment.id= '';
          this.fetchComments();
          this.edit = false;
      }).catch(error => {
         // this.errors.push(error)
         this.comment.body= '';
         this.comment.id= '';
         this.fetchComments();
         this.edit = false;
      });
    },

    deleteComment: function(comment_id){
      axios.delete("../api/post/"+this.currentId+"/comment/"+comment_id)
        .then( function (response){
          this.comment.body= '';
          this.fetchComments();
      }.bind(this));
    },

    showComment: function(comment_id){
      for (var i = 0; i < this.comments.length; i++) {
        if (this.comments[i].id == comment_id) {
          this.comment.body = this.comments[i].body;
          this.comment.id = this.comments[i].id;
          this.edit = true;
        }
      }
    }
  }
}
</script>