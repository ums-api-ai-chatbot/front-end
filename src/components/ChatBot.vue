<template>
    <div class="chat-container">
        <div class="model-selector">
            <label for="model">모델 선택 : </label>
            <select v-model="selectedModel" id="model">
                <option value="basic">기본 모델</option>
                <option value="friendly">친절한 모델</option>
                <option value="formal">공식적인 모델</option>
            </select>
        </div>
        <div class="chat-box" ref="chatBox">
            <div v-for="msg in messages" :key="msg.id" class="message" :class="msg.type">
                <pre>{{ msg.text }}</pre>
            </div>
            <div id="spinner" v-if="isLoading"></div>
        </div>
        <div class="input-area">
            <input 
                type="text" 
                v-model="newMessage" 
                @keyup.enter="sendMessage" 
                placeholder="메시지를 입력하세요..." 
                :disabled="isLoading">
            <button 
                @click="sendMessage" 
                :disabled="isLoading">
                전송
            </button>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            newMessage: '',
            messages: [],
            messageId: 0,
            selectedModel: 'basic', // 기본 선택된 모델
            isLoading: false 
        };
    },
        methods: {
            // 사용자 메시지 전송 메서드
            async sendMessage() {
                if (this.newMessage.trim()) {
                    const userMessage = this.newMessage; // 사용자 메시지 저장
                    this.addMessage(userMessage, 'sent'); // 사용자 메시지 추가
                    this.newMessage = ''; // 입력창 즉시 비우기
                    this.isLoading = true; // 로딩 시작
            
                    try {
                        const response = await this.getBotResponse(userMessage); // 봇 응답 가져오기
                        this.addMessage(response, 'received'); // 봇 메시지 추가
                    } catch (error) {
                        console.error('Error fetching bot response:', error);
                        this.addMessage('응답을 가져오는 중 오류가 발생했습니다.', 'received');
                    } finally {
                        this.isLoading = false; // 로딩 종료
                    }
            
                    this.$nextTick(() => {
                        this.$refs.chatBox.scrollTop = this.$refs.chatBox.scrollHeight; // 스크롤 하단으로 이동
                    });
                }
            },
        // API를 호출하여 봇의 응답을 가져오는 메서드
        async getBotResponse(userMessage) {
            // 여기에 실제 API 엔드포인트를 입력하세요
            // const apiUrl = 'http://211.254.213.18:30000/items';
            const apiUrl = 'http://211.254.213.18:30000/question';
            console.log("userMessage",userMessage)
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    query: userMessage
                }),
            });

            if (!response.ok) {
                throw new Error('Network response was not ok');
            }
            
            //const data = await response.json();
            
            const data = await response.json();
            const res_answer = JSON.stringify(data.answer, null, 2)
            console.log("res_answer",res_answer)
            // console.log("data", JSON.stringify(data, null, 2));
            // return data.answer;

            return res_answer; // 여기에 API 응답에서 봇 메시지를 적절히 추출하는 로직을 입력하세요
        },
        // 메시지 추가하는 헬퍼 메서드
        addMessage(text, type) {
            // 받은 메시지에만 줄바꿈 처리 적용
            let formattedText = text;
            
            if (type === 'received' && typeof text === 'string') {
                // 단순 문자열 치환으로 이스케이프된 줄바꿈 문자를 실제 줄바꿈으로 변환
                formattedText = text.replace(/\\n/g, '\n').replace(/\\"/g, '"');
            }
            
            this.messages.push({ id: this.messageId++, text: formattedText, type });
        },
    },
};
</script>

<style scoped>
.chat-container {
    width: 700px;
    max-width: 100%;
    background: white;
    border-radius: 5px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    overflow: hidden;
}
.model-selector {
    padding: 10px;
    background: #f9f9f9;
    border-bottom: 1px solid #ddd;
}
.chat-box {
    height: 500px;
    overflow-y: auto;
    padding: 10px;
    border-bottom: 1px solid #ddd;
    background-color: #dedede;
}
.message {
    margin: 10px 0;
    padding: 10px;
    border-radius: 5px;
    width: 100%;

}
.message pre {
    margin: 0;
    white-space: pre-wrap; /* 긴 줄이 줄바꿈 되도록 함 */
    font-family: inherit; /* 기본 글꼴 사용 */
    background: transparent; /* 배경 투명하게 */
}
.message.sent {
    float: right;
    background-color: #ffe989;
    text-align: right;
    max-width: 70%; /* 최대 너비를 설정하여 메시지가 너무 넓어지지 않도록 함 */
    display: inline-block; /* 내용에 따라 유동적으로 너비 조정 */
    word-wrap: break-word; /* 긴 단어가 포함된 경우 줄바꿈 처리 */
}
.message.received {
    float: left;
    background-color: #c8c8c8;
    text-align: left;
    max-width: 70%; /* 최대 너비를 설정하여 메시지가 너무 넓어지지 않도록 함 */
    display: inline-block; /* 내용에 따라 유동적으로 너비 조정 */
    word-wrap: break-word; /* 긴 단어가 포함된 경우 줄바꿈 처리 */
}
.input-area {
    padding: 10px;
    display: flex; /* Flexbox로 설정 */
    align-items: center; /* 아이템들을 수직 중앙 정렬 */
}

.input-area input {
    flex: 1; /* 입력 필드가 가능한 공간을 모두 차지하도록 설정 */
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
}

.input-area button {
    padding: 10px;
    border: none;
    border-radius: 5px;
    background-color: #0069d9;
    color: white;
    cursor: pointer;
    margin-left: 10px; /* 버튼과 입력 필드 사이의 여백 */
}

.input-area button:hover {
    background-color: #0056b3;
}
#spinner {
  position: fixed;
  /* margin: calc(50%-25px) auto; */
  bottom:50%;
  left: 49%;
  width: 50px;
  height: 50px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  box-sizing: border-box;
  border-top-color: white;
  border-radius: 100%;

  animation: spin 1s ease-in-out infinite;
}

@keyframes spin {
  100%
  {
    transform: rotate(360deg);
  }
}

</style>
