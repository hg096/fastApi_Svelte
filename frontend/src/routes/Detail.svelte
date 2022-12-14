<script>
    import fastapi from "../lib/api"
    import Error from "../components/Error.svelte"
    import { link, push } from 'svelte-spa-router'
    import { is_login, username } from "../lib/store"
    import { marked } from 'marked'

    import moment from 'moment/min/moment-with-locales'
    moment.locale('ko')


    // Detail 컴포넌트를 호출할때 전달한 파라미터 값을 읽으려면 다음과 같이 params 변수를 선언해야 한다.
    export let params = {}
    let question_id = params.question_id
    // console.log('question_id:'+ question_id)
    // 초기값을 설정해주는 이유는 처음 로드되고 빈값에러를 방지하기 위함 + 초기화
    let question = {answers:[], voter:[], content: ''}
    let content = ""
    let error = {detail:[]}


    function get_question() {
        fastapi("get", `/api/question/detail/${question_id}`, {}, (json) => {
            question = json
        })
    }

    get_question()

    function post_answer(event) {
        event.preventDefault()
        let url = "/api/answer/create/" + question_id
        let params = {
            content: content
        }
        fastapi('post', url, params,
            (json) => {
                content = ''
                error = {detail:[]}
                get_question()
            },
            (err_json) => {
                error = err_json
            }
        )
    }

    function delete_question(_question_id) {
        if(window.confirm('정말로 삭제하시겠습니까?')) {
            let url = "/api/question/delete"
            let params = {
                question_id: _question_id
            }
            fastapi('delete', url, params,
                (json) => {
                    push('/')
                },
                (err_json) => {
                    error = err_json
                }
            )
        }
    }

    function delete_answer(answer_id) {
        if(window.confirm('정말로 삭제하시겠습니까?')) {
            let url = "/api/answer/delete"
            let params = {
                answer_id: answer_id
            }
            fastapi('delete', url, params,
                (json) => {
                    get_question()
                },
                (err_json) => {
                    error = err_json
                }
            )
        }
    }

    function vote_question(_question_id) {
        if(window.confirm('정말로 이 글을 추천하시겠습니까?')) {
            let url = "/api/question/vote"
            let params = {
                question_id: _question_id
            }
            fastapi('post', url, params,
                (json) => {
                    get_question()
                },
                (err_json) => {
                    error = err_json
                }
            )
        }
    }

    function vote_answer(answer_id) {
        if(window.confirm('정말로 댓글을 추천하시겠습니까?')) {
            let url = "/api/answer/vote"
            let params = {
                answer_id: answer_id
            }
            fastapi('post', url, params,
                (json) => {
                    get_question()
                },
                (err_json) => {
                    error = err_json
                }
            )
        }
    }


</script>


<div class="container my-3">
    <!-- 질문 -->
    <h2 class="border-bottom py-2">{question.subject}</h2>
    <div class="card my-3">
        <div class="card-body">
            <!-- style="white-space: pre-line;"  {question.content} -->
            <!-- 스벨트에서 @html은 HTML 코드를 &lt;, &gt; 와 같은 문자열로 변환(strip tags)하지 않고 HTML을 있는 그대로 표시하기 위해 사용 -->
            <div class="card-text" >{@html marked.parse(question.content)}</div>
            <div class="d-flex justify-content-end">
                {#if question.modify_date }
                    <div class="badge bg-light text-dark p-2 text-start mx-3">
                        <div class="mb-2">수정일: </div>
                        <div>{moment(question.modify_date).format("YYYY년 MM월 DD일 hh:mm a")}</div>
                    </div>
                {/if}
                <div class="badge bg-light text-dark p-2">
                    {moment(question.create_date).format("YYYY년 MM월 DD일 hh:mm a")}
                </div>
            </div>
            <div class="my-3">
                <!-- 본인은 본인글에 추천 못하도록 의도 : 태그안에서 if 문으로 처리가 되는지 확인해보기 삼항연산도? -->
                {#if question.user && $username !== question.user.username }
                    <button class="btn btn-sm btn-outline-secondary" on:click="{vote_question(question.id)}">
                        추천<span class="badge rounded-pill bg-success">{ question.voter.length }</span>
                    </button>
                {/if}
                {#if question.user && $username === question.user.username }
                    <a use:link href="/question-modify/{question.id}" class="btn btn-sm btn-outline-secondary">수정</a>
                    <button class="btn btn-sm btn-outline-secondary" on:click={() => delete_question(question.id)}>삭제</button>
                {/if}
            </div>

        </div>
    </div>

    <button class="btn btn-secondary" on:click="{() => {
        push('/')
    }}">목록으로</button>


    <!-- 답변 목록 -->
    <h5 class="border-bottom my-3 py-2">{question.answers.length}개의 답변이 있습니다.</h5>
    {#each question.answers as answer}
    <div class="card my-3">
        <div class="card-body">
            <!-- style="white-space: pre-line;"  {answer.content} -->
            <div class="card-text" >{@html marked.parse(answer.content)}</div>
            <div class="d-flex justify-content-end">
                {#if answer.modify_date }
                    <div class="badge bg-light text-dark p-2 text-start mx-3">
                        <div class="mb-2">수정일: </div>
                        <div>{moment(answer.modify_date).format("YYYY년 MM월 DD일 hh:mm a")}</div>
                    </div>
                {/if}
                <div class="badge bg-light text-dark p-2 text-start">
                    <div class="mb-2">{ question.user ? question.user.username : "유저명 없음"}</div>
                    <div>{moment(question.create_date).format("YYYY년 MM월 DD일 hh:mm a")}</div>
                </div>
            </div>

            <div class="my-3">
                {#if answer.user && $username !== answer.user.username }
                    <button class="btn btn-sm btn-outline-secondary" on:click="{vote_answer(answer.id)}">
                        추천<span class="badge rounded-pill bg-success">{ answer.voter.length }</span>
                    </button>
                {/if}

                {#if answer.user && $username === answer.user.username }
                    <a use:link href="/answer-modify/{answer.id}" class="btn btn-sm btn-outline-secondary">수정</a>
                    <button class="btn btn-sm btn-outline-secondary" on:click={() => delete_answer(answer.id) }>삭제</button>
                {/if}
            </div>
        </div>
    </div>
    {/each}
    <!-- 답변 등록 -->
    <Error error={error} />
    <form method="post" class="my-3">
        <div class="mb-3">
            <textarea rows="10" bind:value={content} disabled={$is_login ? "" : "disabled"} class="form-control" />
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary {$is_login ? '' : 'disabled'}" on:click="{post_answer}" />
    </form>
</div>