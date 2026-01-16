<script setup>
import { ref, onMounted, computed, inject, watch } from 'vue'
import axios from 'axios'
import { ElMessage, ElPagination } from 'element-plus'

const Page = ref({
    Number: 1,
    Size: 5,
    total: 0
})
const Status = inject('Status')
const userName = inject('userName')
const isLiked = ref(false)
const allArticles = ref([])
const filteredArticles = ref([])
const dialogVisible = ref(false)
const currentArticle = ref(null)
const commentContent = ref('')
const articleComments = ref([])

const SearchContent = inject('SearchContent', ref(''))
const registerSearchHandler = inject('registerSearchHandler', () => {})

const currentPageNumber = computed(() => {
    const start = (Page.value.Number - 1) * Page.value.Size
    const end = start + Page.value.Size
    return filteredArticles.value.slice(start, end)
})

const Load = async () => {
    try {
        const response = await axios.get("http://localhost:8000/articles/list")
        if (response.data.code === 1) {
            allArticles.value = response.data.data.list || response.data.data
            filteredArticles.value = [...allArticles.value]//（AI帮助）
            Page.value.total = filteredArticles.value.length
        } else {
            ElMessage({
                message: "加载失败",
                type:"error"
            })
        }
    } catch (e) {
        ElMessage({
            message: "加载失败",
            type:"error"
        })
    }
}

// 简化搜索逻辑（AI帮助）
const handleSearch = (keyword) => {
    if (!keyword) {
        filteredArticles.value = [...allArticles.value]
    } else {
        const key = keyword.toLowerCase()
        filteredArticles.value = allArticles.value.filter(article => {
            return (
                article.title.toLowerCase().includes(key) ||
                article.content.toLowerCase().includes(key) ||
                article.publisher.toLowerCase().includes(key)
            )
        })
    }
    Page.value.Number = 1
    Page.value.total = filteredArticles.value.length
}

const PageChange = (newPage) => {
    Page.value.Number = newPage
}

// 点赞数样式：未点赞=默认色，点赞后=红色[AI帮助]
const likeCountStyle = ref({
    color: "rgb(233,164,58)!important",
    fontWeight: 'normal'
})

// AI帮助
// 加载文章评论列表
const loadComments = async (article_id) => {
    try {
        const res = await axios.get("http://localhost:8000/articles/comments", {
            params: { article_id: article_id }
        })
        if (res.data.code === 1) {
            articleComments.value = res.data.data || []
        } else {
            ElMessage({ message: res.data.msg || "获取失败", type: "error" })
        }
    } catch (e) {
        ElMessage({ message: "获取失败", type: "warning" })
    }
}

// 发表评论（仅登录用户可用）
const submitComment = async () => {
    try {
        const res = await axios.post("http://localhost:8000/articles/comment", {
            article_id: currentArticle.value.id,
            username: userName.value,
            content: commentContent.value.trim()
        })
        if (res.data.code === 1) {
            ElMessage({ message: "发表成功", type: "success" })
            commentContent.value = ''
            loadComments(currentArticle.value.id)
        } else {
            ElMessage({ message: res.data.msg || "发表失败", type: "error" })
        }
    } catch (e) {
        ElMessage({ message: "请求失败", type: "error" })
    }
}

const openDialog = async (article) => {
    currentArticle.value = article
    dialogVisible.value = true
    isLiked.value = false 

    likeCountStyle.value.color = 'rgb(233,164,58)'

    loadComments(article.id)

    if (Status.value === 'success' && userName.value) {
        try {
            const res = await axios.get("http://localhost:8000/articles/check_like", {
                params: {
                    article_id: article.id,
                    username: userName.value
                }
            })
            if (res.data.code === 1) {
                isLiked.value = res.data.is_liked
                if (isLiked.value) {
                    likeCountStyle.value.color = 'red'
                }
            }
        } catch (e) {
            ElMessage({ message: "查询失败", type: "warning" })
        }
    }
}
//AI帮助
const handleLike = async () => {
    if (Status.value !== 'success') {
        ElMessage({ message: "请先登录后再点赞", type: "warning" })
        return
    }

    try {
        if (isLiked.value) {
            const res = await axios.delete("http://localhost:8000/articles/unlike", {
                data: {
                    article_id: currentArticle.value.id,
                    username: userName.value
                }
            })
            if (res.data.code === 1) {
                isLiked.value = false
                currentArticle.value.like_count = res.data.like_count
                likeCountStyle.value.color = 'rgb(233,164,58)'
                ElMessage({ message: "取消点赞成功", type: "success" })
            } else {
                ElMessage({ message: res.data.msg || "取消点赞失败", type: "error" })
            }
        } else {
            const res = await axios.post("http://localhost:8000/articles/like", {
                article_id: currentArticle.value.id,
                username: userName.value
            })
            if (res.data.code === 1) {
                isLiked.value = true
                currentArticle.value.like_count = res.data.like_count
                likeCountStyle.value.color = 'red'
                ElMessage({ message: "点赞成功", type: "success" })
            } else {
                ElMessage({ message: res.data.msg || "点赞失败", type: "error" })
            }
        }
    } catch (e) {
        ElMessage({ message: "点赞失败", type: "error" })
    }
}

onMounted(() => {
    Load()
    // 强制注册搜索函数，确保App能调用（AI帮助）
    registerSearchHandler(handleSearch)

    // 监听关键词变化（AI帮助）
    watch(SearchContent, (newVal) => {
        if (newVal.trim() === '') {
            handleSearch('')
        }
    })
})
</script>

<template>
    <el-text class="text1">Exploration &</el-text>
    <el-text class="text2">Innovation</el-text>
    <p class="text3">分享技术见解，记录开发历程。这是属于程序部的知识库。</p>

    <li class="text4">最新动态</li>
    <div class="card">
        <el-card 
        v-for="article in currentPageNumber" 
        :key="article.id"
        style="width: 80%; margin: 10px auto; background-color: var(--color-background-soft);color:var(--color-f-background-soft)"
        @click="openDialog(article)"
        hover
        >
        <template #header>
            <div class="card-header">
            <span class="title">{{ article.title }}</span>
            </div>
        </template>
            {{ article.content.length > 20 ? article.content.slice(0, 20) + '...' : article.content }}
        <template #footer>
            <div class="card-footer">
            <span class="publisher">发布者：{{ article.publisher }}</span>
            <span style="margin-left: 25px;" class="publish_time">发布时间：{{ article.publish_time }}</span>
            </div>
        </template>
        </el-card>

        <div v-if="currentPageNumber.length === 0 && filteredArticles.length > 0" style="text-align: center; margin: 20px;">
            <el-card style="width: 480px; margin: 10px auto; background-color: var(--color-background-soft);color:var(--color-f-background-soft)" shadow="always">当前页暂无数据</el-card>
        </div>
        <div v-if="filteredArticles.length === 0 && allArticles.length > 0" style="text-align: center; margin: 20px;">
            <el-card style="width: 480px; margin: 10px auto; background-color: var(--color-background-soft);color:var(--color-f-background-soft)" shadow="always">未找到匹配 "{{ SearchContent }}" 的文章</el-card>
        </div>
        <div v-if="allArticles.length === 0" style="text-align: center; margin: 20px;">
            <el-card style="width: 480px; margin: 10px auto; background-color: var(--color-background-soft);color:var(--color-f-background-soft)" shadow="always">暂无文章数据</el-card>
        </div>

        <div style="text-align: center; margin: 20px 0;">
        <el-pagination
            v-model:current-page="Page.Number"
            :page-size="Page.Size"
            :total="Page.total"
            layout="prev, pager, next"
            @current-change="PageChange"
            background
        />
        </div>
    </div>
<el-dialog
    v-model="dialogVisible"
    title=""
    width="80%"
    top="5vh"
    :close-on-click-modal="true"
    :show-close="true"
    style="--el-dialog-bg-color: var(--color-background-soft); --el-dialog-text-color: var(--color-f-background-soft);"
>
    <div v-if="currentArticle" class="dialog-content">
        <h2 class="dialog-title">{{ currentArticle.title }}</h2>
        <div class="dialog-publish">
            <span>发布者：{{ currentArticle.publisher }}</span>
            <span style="margin-left: 20px;">发布时间：{{ currentArticle.publish_time }}</span>
        </div>
        <div class="dialog-content">
            {{ currentArticle.content }}
        </div>

        <span style="margin-left: 20px;">
            <el-button  
                @click="handleLike"
                style="color: RGB(233,164,58); padding: 0;"
            >
                <span :style="likeCountStyle">点赞 ({{ currentArticle.like_count || 0 }})</span>
            </el-button>
        </span>

        <div style="margin-top: 30px; border-top: 1px solid var(--color-f-background-soft); padding-top: 20px;">
            <h3 style="color: rgb(233,164,58); margin-bottom: 15px;">评论区</h3>
            
            <div v-if="Status === 'success'" style="margin-bottom: 20px;">
                <el-input
                    v-model="commentContent"
                    type="textarea"
                    placeholder="请输入评论内容..."
                    style="margin-bottom: 10px; background: var(--color-background-soft); color: var(--color-f-background-soft);"
                    rows="3"
                />
                <el-button 
                    type="primary" 
                    @click="submitComment"
                    style="background: rgb(233,164,58); border: none;"
                >
                    发表评论
                </el-button>
            </div>

            <div v-else style="display: flex;margin-bottom: 20px;align-self: center;justify-self: center; color: red;font-weight: bolder;">
                请先登录后发表评论
            </div>

            <div class="comment-list">
                <div 
                    v-for="(item, index) in articleComments" 
                    :key="index"
                    style="margin-bottom: 15px; padding: 10px; background: rgba(233,164,58,0.1); border-radius: 10px;"
                >
                    <div style="font-weight: bold;color: var(--color-f-background-soft);font-size: x-large;">{{ item.username }}</div>
                    <div style="margin: 5px 0; color: rgb(233,164,58);">{{ item.content }}</div>
                    <div style="font-size: x-small; color: #999;">{{ item.comment_time }}</div>
                </div>
                <div v-if="articleComments.length === 0" style="color: var(--color-f-background-soft);">
                    暂无评论，快来抢沙发～
                </div>
            </div>
        </div>
    </div>
</el-dialog>
</template>

<style scoped>
    .card{
        display:flex ;
        flex-direction: column;
        align-items: center;
        cursor: pointer;
    }

    .text1{
        font-size:xx-large;
        margin-bottom: -20px;
        font-weight: bolder;
        display: block;
        color:var(--color-f-background-soft)
    }

    .text2{
        font-size:xx-large;
        margin-bottom: 10px; 
        font-weight: bolder; 
        display: block; 
        color:RGB(233,164,58);
    }

    .text3{ 
        margin-bottom: 20px;
    }

    .text4{ 
        margin-bottom: 20px; 
        color:RGB(233,164,58); 
    }

    .title{ 
        font-size: x-large; 
        font-weight: bolder; 
        color: rgb(233,164,58);
    }

    .publisher,.publish_time{ 
        font-size: x-small; 
    }

    .article-content { 
        padding: 10px 0; 
        line-height: 1.6; 
    }

    .dialog-content {
        width: 100%;
        padding: 10px 0;
        border-radius: 20px;
    }

    .dialog-title {
        font-size: xx-large;
        font-weight: bolder;
        color: RGB(233,164,58);
        margin-bottom: 15px;
        border-bottom: 1px solid var(--color-f-background-soft);
        padding-bottom: 10px;
    }

    .dialog-publish {
        font-size: x-small;
        color: var(--color-text);
        margin-bottom: 20px;
    }

    .dialog-content {
        color: var(--color-f-background-soft);
        line-height: 1.8;
        white-space: pre-wrap;
        word-wrap: break-word;
    }
</style>