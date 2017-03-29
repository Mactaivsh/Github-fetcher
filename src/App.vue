<template>
	<div id="app">
		<header class="logo">
			<img src="./assets/logo.svg">
			<h1>Github Fetcher</h1>
		</header>
		<section class="search-form" v-loading.fullscreen.lock="Loading">
			<el-form :inline="formInline" :model="searchFormData" :rules="rules" ref="searchForm">
				<el-form-item label="Repositories" prop="repo">
					<el-input v-model="searchFormData.repo" placeholder="repository name"></el-input>
				</el-form-item>
				<el-form-item label="Owner" prop="owner">
					<el-input v-model="searchFormData.owner" placeholder="owner name"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button class="btn-search" icon="search" type="primary" @click="fetchData('searchForm')">Search
					
					</el-button>
				</el-form-item>
			</el-form>
		</section>
		<section class="result" v-show="dataCompleted">
			<el-tabs v-model="tabActiveName" class="result-tab">
				<el-tab-pane label="Info" name="info">
					<el-card>
						<div class="clearfix">
							<div class="owner-info">
								<img :src="avatarUrl" alt="Owner Avatar" class="avatar">
								<p class="repo-name"><span>Name:</span><a :href="repoUrl">{{repoName}}</a></p>
								<p class="owner"><span>Owner:</span><a :href="repoOwnerUrl">{{repoOwner}}</a></p>
								<p class="repo-description">{{repoDescription}}</p>
							</div>
							<div class="repo-detail">
								<p><i class="el-icon-star-on"></i><span>{{repoStars}}</span></p>
								<p><i class="el-icon-information"></i><span>{{repoForks}}</span></p>
								<p><i class="el-icon-share"></i><span>{{repoIssues}}</span></p>
							</div>
						</div>
					</el-card>
				</el-tab-pane>
				<el-tab-pane label="Branches" name="branches">
					<el-card class="card branches-card">
						<div class="branches">
							<div class="branch" v-for="branch in branchesJson">
								<img src="./assets/branch-icon.svg" alt="Branch Icon" class="branch-icon">
								<p class="branch-name"><span>Name:</span> {{branch.name}}</p>
								<p class="branch-url"><span>Url:</span> <a
										:href="repoUrl + '/tree/' + branch.name">{{repoUrl + '/tree/' + branch.name}}</a>
								</p>
							</div>
						</div>
					</el-card>
				</el-tab-pane>
				<el-tab-pane label="Releases" name="releases">
					<el-card class="card releases-card">
						<div class="releases">
							<el-alert
									title="There are no releases yet"
									description="Data successfully retrieved but has no release info "
									type="success"
									show-icon
									:closable="false"
									v-show="isEmptyRelease">
							</el-alert>
							<div class="release" v-for="release in releasesJson">
								<img src="./assets/release-icon.svg" alt="Release Icon" class="release-icon">
								<p class="name"><span>Name:</span>{{release.name}}</p>
								<p class="published-time"><span>Time:</span>{{release['published_at']}}</p>
							</div>
						</div>
					</el-card>
				</el-tab-pane>
			</el-tabs>
		</section>
		<footer class="footer">
			<section class="info">
				<p>Written by <em>Mactavish</em></p>
				<p>Powered by Vue.js</p>
				<p>Theme: Element-UI</p>
			</section>
		</footer>
	</div>
</template>

<script>
    import Axios from "axios"
    Axios.defaults.baseURL = "https://api.github.com"
    export default {
        name: 'app',
        data () {
            return {
                searchFormData: {
                    repo: "",
                    owner: ""
                },
                Loading: false,
                formInline: true,
                tabActiveName: "info",
                dataCompleted: false,
                repoJson: {},
                branchesJson: {},
                releasesJson: {},
                rules: {
                    repo: [
                        { required: true, message: "Missing repository name", trigger: "blur" },
                    ],
                    owner: [
                        { required: true, message: "Missing owner name", trigger: "blur" }
                    ]
                }
            }
        },
        methods: {
            fetchData(formName){
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.Loading = true
//                        Axios({
//                            url: this.getRepoUrl,
//                            method: "get"
//                        }).then(function (response) {
//                            this.Loading = false
//                            this.repoJson = response.data
//                            this.dataCompleted = true
//	                        this.$message.success("Data fetch succeeded")
//                        }.bind(this)).catch(function (err) {
//                            this.Loading = false
//                            this.$message.error("Can not find the Repository")
//                            console.log(err.message)
//                        }.bind(this))
                        Axios.all([this.getRepo(), this.getBranches(), this.getReleases()])
                            .then(Axios.spread(function (repo, branches, releases) {
                                this.Loading = false
                                this.repoJson = repo.data
                                this.branchesJson = branches.data
                                this.releasesJson = releases.data
                                this.$message.success("Data fetch succeeded")
                                this.dataCompleted = true
                            }.bind(this)))
                            .catch(function (err) {
                                this.Loading = false
                                this.$message.error("Data fetch failed")
                                console.log(err.message)
                            }.bind(this))

                    } else {
                        this.$message.warning("Invalid inputs")
                        return false;
                    }
                })

            },
            getRepo() {
                return Axios.get(this.getRepoUrl)
            },
            getBranches() {
                return Axios.get(this.getBranchesUrl)
            },
            getReleases(){
                return Axios.get(this.getReleasesUrl)
            }

        },
        computed: {
            getRepoUrl(){
                return "/repos/" + this.searchFormData.owner.toLowerCase() + "/" + this.searchFormData.repo.toLowerCase()
            },
            getBranchesUrl(){
                return this.getRepoUrl + "/branches"
            },
            getReleasesUrl(){
                return this.getRepoUrl + "/releases"
            },
            avatarUrl(){
                return this.dataCompleted ? this.repoJson.owner["avatar_url"] : ""
            },
            repoUrl(){
                return this.dataCompleted ? this.repoJson["html_url"] : ""
            },
            repoDescription(){
                return this.dataCompleted ? this.repoJson.description : ""
            },
            repoName(){
                return this.dataCompleted ? this.repoJson.name : ""
            },
            repoOwner(){
                return this.dataCompleted ? this.repoJson.owner["login"] : ""
            },
            repoOwnerUrl(){
                return this.dataCompleted ? this.repoJson.owner["html_url"] : ""
            },
            repoForks(){
                return this.dataCompleted ? this.repoJson["forks_count"] : 0
            },
            repoStars(){
                return this.dataCompleted ? this.repoJson["stargazers_count"] : 0
            },
            repoIssues(){
                return this.dataCompleted ? this.repoJson["open_issues_count"] : 0
            },
            isEmptyRelease(){
                return Object.keys(this.releasesJson).length <= 0
            }
        }
        ,
        beforeMount()
        {
            if (document.documentElement.clientWidth < 768) {
                this.formInline = false
            }
        }
    }
</script>

<style>
	
	body {
		background-color: #F9FAFC;
	}
	
	.clearfix::after,
	.clearfix::before {
		content: "";
		display: table;
		
	}
	
	.clearfix::after {
		clear: both;
	}
	
	#app {
		font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #1F2D3D;
		margin-top: 60px;
	}
	
	.logo {
		max-width: 1100px;
		width: 100%;
		margin: auto;
		text-align: center;
	}
	
	.logo img {
		width: 150px;
	}
	
	.logo h1 {
		font-weight: 200;
		color: #C0CCDA;
	}
	
	@media screen and (max-width: 767px) {
		.logo img {
			width: 80px;
		}
		
		.logo h1 {
			font-weight: 200;
			font-size: 20px;
			color: #C0CCDA;
		}
	}
	
	.search-form {
		width: 100%;
		margin: auto;
		font-weight: 300;
	}
	
	.search-form .btn-search {
		background-color: #1D8CE0;
	}
	
	.result {
		width: 600px;
		margin: auto;
		font-weight: 300;
	}
	
	@media screen and (max-width: 767px) {
		.result {
			width: 100%;
			margin: auto;
		}
		
		.btn-search {
			width: 100%;
		}
	}
	
	.owner-info {
		width: 80%;
		float: left;
	}
	
	.owner-info .avatar {
		display: block;
		width: 60px;
		margin-right: 20px;
		float: left;
	}
	
	.owner-info > p {
		margin: 0;
		line-height: 20px;
		text-align: left;
		vertical-align: middle;
		text-overflow: ellipsis;
	}
	
	.owner-info > p span {
		color: #1D8CE0;
	}
	
	.owner-info > p > a {
		color: #1F2D3D;
		text-decoration: none;
	}
	
	.owner-info > p > a:hover {
		text-decoration: underline;
	}
	
	.owner-info .repo-description {
		font-size: 13px;
		color: #99A9BF;
		overflow: hidden;
		text-overflow: ellipsis;
	}
	
	.repo-detail {
		width: 20%;
		float: right;
	}
	
	.repo-detail > p {
		margin: 0;
		line-height: 20px;
		text-align: right;
		vertical-align: middle;
		font-size: 12px;
	}
	
	.repo-detail > p span {
		margin-left: 5px;
	}
	
	.branches {
		width: 100%;
	}
	
	.branches .branch + .branch {
		margin-top: 15px;
	}
	
	.branches .branch .branch-icon {
		display: block;
		float: left;
		width: 40px;
		margin-right: 20px;
	}
	
	.branches .branch > p {
		margin: 0;
		line-height: 20px;
		text-align: left;
		overflow: hidden;
		vertical-align: middle;
		text-overflow: ellipsis;
	}
	
	.branches .branch > p span {
		color: #1D8CE0;
	}
	
	.branches .branch .branch-url a {
		text-decoration: none;
		color: #1F2D3D;
		font-size: 14px;
	}
	
	.branches .branch .branch-url a:hover {
		text-decoration: underline;
	}
	
	.releases {
		width: 100%;
	}
	
	.releases .release + .release {
		margin-top: 15px;
	}
	
	.releases .release .release-icon {
		display: block;
		float: left;
		width: 40px;
		margin-right: 20px;
	}
	
	.releases .release > p {
		margin: 0;
		line-height: 20px;
		text-align: left;
		overflow: hidden;
		vertical-align: middle;
		text-overflow: ellipsis;
	}
	
	.releases .release > p span {
		color: #1D8CE0;
	}
	
	.footer {
		margin-top: 30px;
		color: #8492A6;
		font-size: 14px;
		font-weight: 300;
	}
</style>
