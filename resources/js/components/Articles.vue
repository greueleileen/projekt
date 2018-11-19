<template>
    <div>
        <div class="bg-card-header card-header mb-4">Blog Einträge</div>
        <div class="mb-2">
            <div class="card-header login_font bg-card-header2">Füge einen neuen Eintag hinzu.</div>
            <form @submit.prevent="addArticle" class="mb-3 form-border padding-form">
                <div class="form-group">
                    <input type="text" class="form-control " placeholder="Titel" v-model="article.title">
                </div>
                <div class="form-group">
                    <textarea class="form-control" placeholder="Artikelbeschreibung" v-model="article.body"></textarea>
                </div>
                <button type="submit" class="btn btn-block btn-custom-primary">Eintrag Speichern</button>
            </form>
        </div>
        <div class="card-border mb-3" v-for="article in articles" v-bind:key="article.id">
            <div class="card-header login_font bg-card-header3">{{ article.title}}</div>
            <div class="card-body mb-2 article-font">{{ article.body}}</div>
            <div class="card-padding">
                <button class="btn btn-delete btn-block mb-2" @click="deleteArticle(article.id)">Artikel löschen</button>
                <button class="btn btn-update btn-block mb-2" @click="editArticle(article)">Artikel bearbeiten</button>
            </div>
        </div>
        <nav aria-label="Pagination">
            <ul class="pagination">
                <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item">
                    <a class="page-link bg-page-btn" href="#" tabindex="-1" @click="fetchArticles(pagination.prev_page_url)"><</a>
                </li>
                <li class="page-item disabled"><a class="page-link text-dark" href="#">Seite {{ pagination.current_page }} von {{ pagination.last_page }}</a></li>

                <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item">
                    <a class="page-link bg-page-btn" href="#" @click="fetchArticles(pagination.next_page_url)">></a>
                </li>
            </ul>
        </nav>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                articles: [],
                article: {
                    id: '',
                    title: '',
                    body: ''
                },
                article_id: '',
                pagination: {},
                edit: false
            }
        },

        created(){
            this.fetchArticles();
        },

        methods: {
            fetchArticles(page_url){
                let vm = this;
                page_url =page_url || '/api/articles';
                fetch(page_url)
                    .then(res => res.json())
                    .then(res => {
                        this.articles = res.data;
                        vm.makePagination(res.meta, res.links)
                    })
                    .catch(err => console.log(err));
            },
            makePagination(meta, links){
                let pagination = {
                    current_page: meta.current_page,
                    last_page: meta.last_page,
                    next_page_url: links.next,
                    prev_page_url: links.prev
                }
                this.pagination = pagination;
            },
            deleteArticle(id) {
                if(confirm('Bist du dir sicher, dass du diesen Artikel löschen möchtest?')){
                     fetch(`api/article/${id}` ,{
                         method: 'delete'
                    })
                    .then(res => res.json())
                         .then(data => {
                             alert('Artikel gelöscht');
                             this.fetchArticles();
                         })
                         .catch(err => console.log(err));
                }
            },
            addArticle() {
                if(this.edit === false) {
                    //Add new article
                    fetch('api/article', {
                        method: 'post',
                        body: JSON.stringify(this.article),
                        headers: {
                            'content-type': 'application/json'
                        }
                    })
                        .then(res => res.json())
                        .then(data => {
                            this.article.title = '';
                            this.article.body = '';
                            alert('Artikel hinzugefügt');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                } else {
                    //Update article
                    fetch('api/article', {
                        method: 'put',
                        body: JSON.stringify(this.article),
                        headers: {
                            'content-type': 'application/json'
                        }
                    })
                        .then(res => res.json())
                        .then(data => {
                            this.article.title = '';
                            this.article.body = '';
                            alert('Artikel geändert');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                }
            },
            editArticle(article) {
                this.edit = true;
                this.article.id = article.id;
                this.article.article_id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;
            }
        }
    };
</script>

<style scoped>

</style>