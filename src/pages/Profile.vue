<template>
    <div class="profile">
        <div class="column-1">
            <div class="avatar">
                <label>
                    <img :src="avatar" :alt="username">
                    <span>Upload</span>
                    <input type="file" @change="upload">
                </label>
                <button @click="editProfile">Edit</button>
            </div>
            <div class="followers" v-show="showFollowers">
                <span>Followers</span>
                <div>
                    <div class="item" v-for="(item, id) in users" :key="`user_item_${id}`">
                        <div class="img">
                            <router-link :to="`/users/${item.id}`">
                                <img :src="item.avatar || noImage" alt=""/>
                                <span>{{ item.firstName }}</span>
                            </router-link>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="column-2">
            <div class="about">
                <h1>{{ firstName }} {{ lastName }}</h1>
                <div>
                    <div>
                        <h3>Birthday:</h3>
                        <span>{{ birthday }}</span>
                    </div>
                    <div v-show="this.hometown">
                        <h3>Hometown:</h3>
                        <span>{{ hometown }}</span>
                    </div>
                    <div v-show="this.language">
                        <h3>Language:</h3>
                        <span>{{ language }}</span>
                    </div>
                </div>
            </div>
            <div class="posts" v-show="showPosts">
                <div>
                    <div class="head">
                        <span>All posts</span>
                    </div>
                    <div class="item" v-for="(item, id) in userPosts" :key="`userPosts_item_${id}`">
                        <div class="author">
                            <div>
                                <img :src="item.avatar || noImage" alt=""/>
                                <span>{{ item.firstName }} {{ item.lastName }}</span>
                            </div>
                            <span>{{ item.time }}</span>
                        </div>
                        <div class="text">
                            <span>{{ item.text }}</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'Profile',
        data() {
            return {
                firstName: '',
                lastName: '',
                birthday: '',
                username: '',
                email: '',
                avatar: '',
                hometown: '',
                language: '',
                isEdit: false,
                needCheck: false,
                noImage: '//localhost/medium/src/assets/user.png',
                users: [],
                userPosts: [],
                showFollowers: true,
                showPosts: true
            }
        },
        methods: {
            upload(event) {
                console.log(event);
                const data = new FormData();
                data.append('file', event.target.files[0]);
                data.append('userId', localStorage.getItem('id'));

                fetch(
                    '//localhost/medium/api/user/user.php',
                    {
                        method: 'post',
                        body: data
                    }
                ).then(res => res.json()).then(res => {
                    if(res.res) {
                        this.avatar = res.src;
                        location.reload(true);
                    }
                });
            },
            editProfile() {
                if(this.isEdit === false) {
                    this.isEdit = true;
                    this.$router.push('/edit');
                }
            }
        },
        mounted() {
            fetch(
                '//localhost/medium/api/user/user.php?id='+localStorage.getItem('id'), 
                {
                    method: 'get',
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }
            ).then(res => res.json()).then(res => {
                console.log(res);

                const date = new Date(+res.user.birthday * 1000);
                const month = '' + (date.getMonth() + 1);
                const day = '' + date.getDate();
                let strDate = (day.length < 2 ? '0' + day : day) + '.' + (month.length < 2 ? '0' + month : month) + '.' + date.getFullYear();

                this.firstName = res.user.firstName;
                this.lastName = res.user.lastName;
                this.birthday = strDate;
                this.hometown = res.user.hometown;
                this.language = res.user.language;
                this.avatar = res.user.avatar || '//localhost/medium/src/assets/user.png';
            });

            fetch(
                '//localhost/medium/api/user/followers.php?id='+localStorage.getItem('id'), 
                {
                    method: 'get',
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }
            ).then(res => res.json()).then(res => {
                console.log(res, 'followers');
                if(res.users.length === 0) {
                    console.log('пусто');
                    this.showFollowers = false;
                } else {
                    this.users = res.users || [];
                }
            });

            fetch(
                '//localhost/medium/api/user/posts.php?id='+localStorage.getItem('id'),
                {
                    method: 'get',
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }
            ).then(res => res.json()).then(res => {
                console.log(res);
                if(res.userPosts.length === 0) {
                    console.log('пусто');
                    this.showPosts = false;
                } else {
                    this.userPosts = res.userPosts || [];
                }
            });
        }
    }
</script>

<style lang="scss" scoped>
    .profile {
        width: 70%;
        margin: 0 auto;
        display: flex;
        justify-content: space-between;

        .column-1 {
            width: 25%;
            display: flex;
            flex-direction: column;

            .avatar {
                height: auto;
                display: flex;
                flex-direction: column;
                justify-content: space-between;
                box-sizing: border-box;
                border-radius: 4px;
                padding: 1rem;
                margin-right: 1rem;
                box-shadow: 0 0 4px 1px lightgrey;
                margin-bottom: 1rem;

                label {
                    width: auto;
                    height: auto;
                    overflow: hidden;
                    position: relative;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    margin: auto 0;
                    border-radius: 4px;
                    box-shadow: 0 0 8px 2px lightgrey;
                    background-color: white;

                    img {
                        width: 100%;
                        height: 100%;
                        object-fit: cover;
                    }

                    span {
                        display: flex;
                        opacity: 0;
                        position: absolute;
                        width: 100%;
                        height: 100%;
                        align-items: center;
                        justify-content: center;
                        background-color: rgba($color: #0f63c4, $alpha: .5);
                        font-size: 1.1rem;
                        transition: .5s;
                        color: white;
                        font-weight: 600;
                        border-radius: 4px;
                        left: 0;
                        top: 0;
                    }
                                
                    input {
                        display: none;
                    }

                    &:hover {
                        cursor: pointer;

                        span {
                            opacity: 1;
                            transition: .5s;
                        }
                    }
                }

                button {
                    padding: .3rem 2.5rem;
                    margin-top: 1rem;
                    border: none;
                    border-radius: 6px;
                    font-size: 1rem;
                    background-color: #1074e7;
                    color: white;
                    font-weight: 500;
                    line-height: 20px;
                    transition: .3s;

                    &:hover {
                        transition: .3s;
                        background-color: #0f63c4;
                    }

                    &:active {
                        outline: none;
                        transition: box-shadow .15s;
                        box-shadow: 0px 0px 4px 2px #5aa0f1;
                    }

                    &:focus {
                        outline: none;
                    }
                }
            }

            .followers {
                height: auto;
                display: flex;
                flex-direction: column;
                justify-content: space-between;
                box-sizing: border-box;
                border-radius: 4px;
                padding: 1rem;
                margin-right: 1rem;
                box-shadow: 0 0 4px 1px lightgrey;

                > span {
                    font-size: 1rem;
                    font-weight: 300;
                    margin-bottom: 11px;
                    padding-bottom: 10px;
                    border-bottom: 1px solid lightgrey;
                }

                > div {
                    display: flex;
                    justify-content: space-evenly;
                    align-items: center;
                    flex-wrap: wrap;

                    .item {
                        margin: 5px;
                        width: 25%;
                        
                        .img {
                            a {
                                display: flex;
                                flex-direction: column;
                                align-items: center;
                                text-decoration: none;

                                img {
                                    width: 50px;
                                    height: 50px;
                                    border-radius: 50%;
                                    object-fit: cover;
                                }

                                span {
                                    font-size: 1rem;
                                    font-weight: 300;
                                    color: black;

                                    &:hover {
                                        text-decoration: underline;
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }

        .column-2 {
            width: 75%;
            display: flex;
            flex-direction: column;

            .about {
                height: auto;
                display: flex;
                flex-direction: column;
                justify-content: space-between;
                box-sizing: border-box;
                border-radius: 4px;
                padding: 1rem;
                box-shadow: 0 0 4px 1px lightgrey;
                margin-bottom: 1rem;

                h1 {
                    margin: 0;
                    padding: 0;
                    font-size: 1.5rem;
                    font-weight: 300;
                    margin-bottom: 1rem;
                    padding-bottom: 1rem;
                    border-bottom: 1px solid lightgrey;
                }

                > div {
                    > div {
                        display: flex;
                        align-items: center;
                        
                        &:not(:first-child) {
                            margin-top: 10px;
                        }

                        h3 {
                            width: 30%;
                            margin: 0;
                            padding: 0;
                            font-size: 1.1rem;
                            font-weight: 300;
                        }

                        span {
                            font-size: 1.1rem;
                            font-weight: 500;
                        }
                    }
                }
            }

            .posts {
                width: 100%;
                height: auto;

                .head {
                    width: 100%;
                    height: auto;
                    display: flex;
                    flex-direction: column;
                    box-sizing: border-box;
                    border-radius: 4px;
                    padding: 1rem;
                    box-shadow: 0 0 4px 1px lightgrey;
                    margin-bottom: 1rem;

                    span {
                        font-weight: 300;
                        font-size: 1rem;
                    }
                }

                .item {
                    width: 100%;
                    height: auto;
                    display: flex;
                    flex-direction: column;
                    box-sizing: border-box;
                    border-radius: 4px;
                    padding: 1rem;
                    box-shadow: 0 0 4px 1px lightgrey;
                    margin-bottom: 1rem;

                    .author {
                        display: flex;
                        justify-content: space-between;
                        align-items: center;
                        margin-bottom: 1rem;
                        padding-bottom: 1rem;
                        border-bottom: 1px solid lightgrey;

                        div {
                            display: flex;
                            justify-content: center;
                            align-items: center;

                            img {
                                width: 45px;
                                height: 45px;
                                object-fit: cover;
                                border-radius: 50%;
                            }

                            span {
                                margin-left: 10px;
                                font-weight: 500;
                                font-size: 1rem;
                            }
                        }
                    }
                }
            }
        }
    }
</style>