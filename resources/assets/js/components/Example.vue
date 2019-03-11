<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <div class="panel panel-default">
                    <div class="panel-heading">Example Vue Component</div>

                    <div class="panel-body">
                        I'm an example component!
                        <h4>Tags: </h4>
                        <ul>
                        <li v-for="(tag, index) in tags">
                            {{ tag }}
                        </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
      data () {
        return {
          tags: null
        }
      },
      mounted () {
        axios
          .get('/api/tags')
          .then(response => (this.tags = response && response.status == 200? response.data.tags:[]))
      }
    }
</script>
