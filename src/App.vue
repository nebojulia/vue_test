<template>
    <div class="app">
        <h1>Страница с постами</h1>
        <my-input
            v-model="searchQuery"
            placeholder="Поиск..."
        />
        <div class="app__btns">
            <my-button 
                @click="showDialog" 
                >
                Создать пост
            </my-button>
            <my-select
                v-model="selectedSort"
                :options="sortOptions"
            />
        </div>
        <my-dialog v-model:show="dialogVisible">
            <post-form
                @create = "createPost"
            />
        </my-dialog>
        <post-list 
            :posts="sortedAndSearchedPost"
            @remove="removePost"
            v-if="!isPostLoading"
        />
        <div v-else>Идёт загрузка...</div>
        <div ref="observer" class="observer"></div>
    <!--<div class="page__wrapper">
            <div 
                v-for="pageNum in totalPages" 
                :key="pageNum"
                :class="{
                    'current-page': page === pageNum
                }"
                @click="changePage(pageNum)"
            >
                    {{ pageNum }}
            </div>
        </div>-->
    </div>
</template>

<script>
import PostForm from "@/components/PostForm";
import PostList from "@/components/PostList";
import MyButton from "@/components/UI/MyButton";
import MySelect from "@/components/UI/MySelect";
import MyDialog from "@/components/UI/MyDialog";
import MyInput from "@/components/UI/MyInput";
import axios from "axios";
import { timingSafeEqual } from "crypto";

export default {
    components: {
        PostList, 
        PostForm, 
        MyButton, 
        MySelect, 
        MyDialog, 
        MyInput
    },
    data() {
        return {
            posts: [],
            dialogVisible: false,
            isPostLoading: false,
            selectedSort: '',
            searchQuery: '',
            page: 1,
            limit: 10,
            totalPages: 0,
            sortOptions: [
                {value: 'title', name: 'По названию'},
                {value: 'body', name: 'По содержимому'}
            ]
        }
    },
    methods: {
        createPost(post) {
            this.posts.push(post);
            this.dialogVisible = false;
        },
        removePost(post) {
            this.posts = this.posts.filter(p => p.id !== post.id)
        }, 
        showDialog() {
            this.dialogVisible = true;
        },
        //changePage(pageNum) {
        //    this.page = pageNum;
        //},
        async fetchPosts() {
            try {
                this.isPostLoading = true;
                const response = await axios.get('https://jsonplaceholder.typicode.com/posts', {
                    params: {
                        _page: this.page,
                        _limit: this.limit,
                    }
                });
                this.totalPages = Math.ceil(response.headers['x-total-count'] / this.limit)
                this.posts = response.data;
            } catch (e) {
                alert('Ошибка')
            } finally {
                this.isPostLoading = false;
            }
        },
        async loadMorePosts() {
            try {
                this.page += 1;
                const response = await axios.get('https://jsonplaceholder.typicode.com/posts', {
                    params: {
                        _page: this.page,
                        _limit: this.limit,
                    }
                });
                this.totalPages = Math.ceil(response.headers['x-total-count'] / this.limit)
                this.posts = [...this.posts, ...response.data];
            } catch (e) {
                alert('Ошибка')
            }
        }
    },
    mounted() {
        this.fetchPosts();
        console.log(this.$refs.observer);
        const options = {
            rootMargin: '0px',
            threshold: 1.0
        }
        const callback = (entries, observer) => {
            if (entries[0].isIntersecting && this.page < this.totalPages) {
                this.loadMorePosts();
            }
        }
        const observer = new IntersectionObserver(callback, options);
        observer.observe(this.$refs.observer);
    },
    computed: {
        sortedPost() {
            return[...this.posts].sort((post1, post2) => post1[this.selectedSort]?.localeCompare(post2[this.selectedSort]))
        },
        sortedAndSearchedPost(){
            return this.sortedPost.filter(post => post.title.toLowerCase().includes(this.searchQuery.toLowerCase()))
        }
    },
    watch: {
        //page() {
        //    this.fetchPosts();
        //}
    }
}
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

.app {
    padding: 20px;
}
.app__btns {
    margin: 15px 0;
    display: flex;
    justify-content: space-between;
}
.page__wrapper {
    display: flex;
    margin: 15px auto 0;
    width: 150px;
    justify-content: space-between;
}
.page {
    border: 1px solid black;
    padding: 10px;
}
.current-page {
    border: 2px solid teal;
}
</style>