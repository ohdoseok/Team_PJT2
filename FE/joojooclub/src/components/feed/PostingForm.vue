<template>
<div>
	<div class="posting-box border">
		<h3 v-if="action === `create`" class="mb-5">피드 작성</h3>
		<h3 v-if="action === `update`" class="mb-5">피드 수정</h3>
		<search-bar v-if="isDrinkNames" :drinkNames="drinkNames" class="search-form"></search-bar>
		<form class="ui form" enctype="multipart/form-data">

			<div class="field">
				<label for="drinkSearch">전통주명</label>
				<input type="text" v-model="selectedDrinkName" class="form-control" readonly>
			</div>
			<span v-if="drinkError" class="sub-error">* 전통주명을 검색해주세요.</span>

			<div class="field">
				<label for="title">제목</label>
				<input v-model.trim="newFeed.title" type="text" multiple="multiple" id="title" placeholder="제목 (15자 미만)" class="form-control">
			</div>
			<span v-if="titleError" class="sub-error">* 제목을 입력해주세요.</span>
			<span v-if="maxTitleError" class="sub-error">* 15자 미만으로 입력해주세요. (현재 : {{ newFeed.title.length }}자)</span>
			
			<div class="field">
				<label for="content">내용</label>
				<textarea v-model.trim="newFeed.content" id="content" placeholder="내용 (255자 이하)"></textarea>
			</div>
			<span v-if="contentError" class="sub-error">* 내용을 입력해주세요.</span>
			<span v-if="maxContentError" class="sub-error">* 255자 이내로 입력해주세요. (현재 : {{ newFeed.content.length }}자)</span>

			<div class="field">
				<label for="imageFile">첨부 파일</label>
				<div class="col">
					<div class="d-flex col">
						<input type="file" accept="image/*" ref="image" id="imageFile" @change="uploadImage($event)">
						<div v-if="this.newFeed.previewImgUrl" class="btn colorbtn btn-outline-warning" @click="onDeleteImage()">업로드 취소</div>
					</div>
					<img v-if="newFeed.previewImgUrl" :src="newFeed.previewImgUrl" alt="uploaded feed image" class="preview-image p-4" style="width: 50vw; max-width: 30rem; min-width: 220px">
				</div>
			</div>

			<div class="field">
				<label for="tags">태그</label>
				<input v-model.trim="newFeed.customTags" type="text" id="tags" placeholder="태그">
				<span v-if="maxTagsError" class="sub-error">* 20자 이내로 입력해주세요. (현재 : {{ newFeed.customTags.length }}자)</span>
			</div>

			<button class="btn colorbtn2 btn-white mt-3" @click="goback">뒤로</button>
			<button class="btn colorbtn2 btn-warning mt-3" @click.prevent="onSubmit">저장</button>
		</form>
	</div>
</div>
</template>

<script>
import SearchBar from '@/components/feed/SearchBar'
import { mapActions, mapGetters } from 'vuex'

export default {
	name: 'PostingForm',
	components: {
		SearchBar,
	},
	props: {
		feed: Object,
		action: String,
		currentUser: Object,
	},
	data() {
		return {
			newFeed: {
				feedIndex: this.feed.feedIndex,
				drinkIndex: this.feed.drink.drinkIndex,
				drinkName: this.feed.drink.drinkName,
				title: this.feed.title,
				content: this.feed.content,
				customTags: this.feed.customTags,
				imgFile: this.feed.imgFile ? this.feed.imgFile : '',
				imgUrl: this.feed.imageUrl ? this.feed.imageUrl : '',
				previewImgUrl: this.feed.imageUrl	// 업로드한 이미지 미리보기
			},
			memberIndex: this?.currentUser?.member?.memberIndex,
			
			// required error 처리
			drinkError: false,
			titleError: false,
			contentError: false,
		}
	},
	computed: {
		...mapGetters('drinks', ['isSearched', 'searchedDrink', 'isDrinkNames', 'drinkNames']),
		selectedDrinkName() {
			if (this.isSearched) { 
				this.setSearchDrink({drinkIndex: this.feed.drink.drinkIndex, drinkName: this.feed.drink.drinkName})
				return this.feed.drink.drinkName
			} else { 
				return this.searchedDrink.drinkName
			}
		},
		// validation check
		maxTitleError() {
			if (this.newFeed.title.length >= 15) return true
			else return false
		},
		maxContentError() {
			if (this.newFeed.content.length >= 255) return true
			else return false
		},
		maxTagsError() {
			if (this.newFeed.customTags.length >= 20) return true
			else return false
		},
	},
	methods: {
		...mapActions('feed', ['createFeed','updateFeed']),
		...mapActions('drinks', ['fetchDrinkNames', 'setSearchDrink']),
		goback() {
			this.$router.push({
				name: 'profile',
				params: {
					userPK: this?.memberIndex
				}
			})
		},
		checkRequiredError(val) {
			if (val === undefined) return true
			else if (val === null) return true
			else if (val === '') return true
			else return false
		},
		onSubmit() {
			// required 확인
			this.drinkError = this.checkRequiredError(this.searchedDrink.drinkIndex)
			this.titleError = this.checkRequiredError(this.newFeed.title)
			this.contentError = this.checkRequiredError(this.newFeed.content)
			if (!this.drinkError && !this.titleError && !this.contentError && !this.maxTitleError && !this.maxContentError && !this.maxTagsError) {
				// form data 선언
				let formdata = new FormData()
				// 키값 추가
				this.newFeed.drinkIndex = this.searchedDrink.drinkIndex,
				formdata.append('drinkIndex', this.newFeed.drinkIndex)
				formdata.append('title', this.newFeed.title)
				formdata.append('content', this.newFeed.content)
				formdata.append('customTags', this.newFeed.customTags)

				if (this.action === 'create') {
					formdata.append('imgFile', this.newFeed.imgFile)
					this.createFeed(formdata)
				} else if (this.action === 'update') {
					formdata.append('feedIndex', this.feed.feedIndex)
					formdata.append('imgFile', this.newFeed.imgFile)
					formdata.append('imgUrl', this.newFeed.imgUrl)
					// for (let [k, v] of formdata.entries()) console.log(k, v)
					this.updateFeed(formdata)
				}
			}
		},
		uploadImage(event) {
			// 업로드한 이미지 파일 가져오기
			this.newFeed.imgFile = event.target.files[0]
			// 기존 이미지 url 필드 삭제
			this.newFeed.imgUrl = ''	
			// 미리보기를 위한 url 변환
			this.newFeed.previewImgUrl = URL.createObjectURL(this.newFeed.imgFile)
		},
		onDeleteImage() {		// 업로드한 이미지 삭제
			this.newFeed.imgFile = ''
			this.newFeed.imgUrl = ''
			this.newFeed.previewImgUrl = ''
			// 업로드 파일 배열 초기화
			this.$refs.image.value = ''
		},
	},
	watch: {
		feed() {
			this.newFeed = this.feed
		},
	},
	created() {
		this.fetchDrinkNames()
	},
}
</script>

<style scoped>
* {
	font-family: 'NanumSquareRound';
}
.posting-box {
	padding: 2rem 5vw 2rem 0;
	border-radius: 20px;
	display: flex;
	flex-direction: column;
}
.ui.form textarea {
	line-height: 1.8;
}
.ui.form .field {
	margin-top: 1rem;
}
.field {
	display: grid;
	column-gap: 2vw;
	grid-template-columns: 10vw auto;
}
.field > label {
	text-align: end;
	/* word-break: keep-all; */
}
.search-form {
	width: 50%;
	margin-bottom: 1.5vw;
	margin-left: 25vw;
}
#imageFile {
	flex: 1;
}
.colorbtn {
  font-size: min(2vw, 1rem);
  padding: min(0.8vw, 4px) min(1.6vw, 1rem);
  margin: 5px;
  color: black;
}
.colorbtn2 {
  font-size: min(2vw, 1rem);
  padding: min(0.8vw, 4px) min(1.6vw, 1rem);
  margin: 5px;
}
.btn-outline-warning {
  color: black;
  border-color: rgb(237, 198, 150);
  border-radius: 5px;
}

.btn-outline-warning:hover,
.btn-outline-warning:focus,
.btn-outline-warning:active {
  background-color: rgb(233, 187, 131);
  color: black;
  border-color: rgb(233, 187, 131);
}
</style>