<template>
    <div id="app">
        <div id="container">
            <div class="row">
                <div class="col-md-6 col-md-offset-3">
                    <h1 class="header">{{ msg }}</h1>
                    <form id="changeRepoForm" @submit.prevent="changeRepo()" class="form-inline">
                        <div class="form-group">
                            <label for="fullRepoName">Full Repo Name</label>
                            <input type="text" name="fullRepoName" v-model="fullRepoName" class="form-control">
                        </div>
                        <input type="submit" class="btn btn-primary" value="Get repo filesystem!">
                    </form>
                    <hr>
                    <table class="table">
                        <caption>{{ path }}</caption>
                        <thead>
                            <tr>
                                <th>Name</th>
                                <th class="text-right">
                                    <button class="btn btn-success btn-xs" @click="goBack()" v-show="path !== '/'">Go Back</button>
                                </th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="file in sortedFiles">
                                <td>
                                    <div class="file" v-if="file.type === 'file'">
                                        <i class="fa fa-file-o"></i> {{ file.name }}
                                    </div>
                                    <div class="directory" v-if="file.type === 'dir'">
                                        <i class="fa fa-folder-o"></i>
                                        <a @click="changePath(file.path)">{{ file.name }}</a>
                                    </div>
                                </td>
                                <td class="text-right">
                                    <a :href="file.download_url" download v-if="file.type === 'file'">
                                        <i class="fa fa-cloud-download"></i>
                                    </a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import axios from 'axios';

export default {
    name: 'app',
    data() {
        return {
            msg: 'Welcome to Github Explorer!',
            fullRepoName: '',
            username: '',
            repo: '',
            path: '/',
            files: [],
            errors: []
        }
    },
    methods: {
        changeRepo: function() {
            var splitData = this.fullRepoName.split('/');
            this.username = splitData[0];
            this.repo = splitData[1];
            this.path = '/';
            console.group("Vue Data");
            console.log("fullRepoName:", this.fullRepoName);
            console.log("username:", this.username);
            console.log("repo:", this.repo);
            console.groupEnd("Vue Data");
            this.getFiles();
        },
        getFiles: function() {
            axios.get('https://api.github.com/repos/' + this.fullRepoUrl + '/contents' + this.path)
                .then(response => {
                    this.files = response.data
                })
                .catch(e => {
                    this.errors.push(e.response);
                })

        },
        changePath: function(path) {
            this.path = '/' + path;
            this.getFiles();
        },
        goBack: function() {
            this.path = this.path.split('/').slice(0, -1).join('/');
            if (this.path === '') this.path = '/';

            this.getFiles();
        }
    },
    computed: {
        fullRepoUrl: function() {
            return this.username + '/' + this.repo;
        },
        sortedFiles: function() {
            return this.files.slice(0).sort(function(a, b) {
                if (a.type !== b.type) {
                    if (a.type === 'dir') {
                        return -1;
                    } else {
                        return 1;
                    }
                } else {
                    if (a.name < b.name) {
                        return -1;
                    } else {
                        return 1;
                    }
                }
            });
        }
    }
}
</script>
<style>
#app {
    margin-top: 60px;
}

.header {
    margin-bottom: 30px;
}
</style>
