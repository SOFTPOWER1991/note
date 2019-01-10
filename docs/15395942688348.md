public interface SearchApi {
    @FormUrlEncoded
    @POST(ProjectRetrofitApi.Suggestion_PROJECTS_URL)
    Observable<BaseRetrofitModel<Object>> getSuggestionProjectsList(
            @FieldMap Map<String, Object> map);


    /**
     * 人脉搜索接口
     *
     * @param page
     * @param map
     * @return
     */
    @FormUrlEncoded
    @POST(ProjectRetrofitApi.FRIEND_SEARCH_RELATIONS + "/{page}")
    Observable<BaseRetrofitModel<Object>> getFriendSearchRelationsListRx(
            @Path("page") String page,
            @FieldMap Map<String, Object> map);


public final static String Suggestion_PROJECTS_URL = "s/project/getSuggestion";
    /**
     * 人脉搜索
     */
    public final static String FRIEND_SEARCH_RELATIONS = "s/friend/searchRelations";
    
    
    /**
         * 检查好友关系状态接口
         */
        fun checkRelationStatus(accessToken: String, friendId: String, callback: ApiResponseCallback<BaseRetrofitModel<Any>>) {
            var map: MutableMap<String, Any> = CommonUtils.createMap()

            map[ACCESS_TOKEN] = accessToken
            map["friendId"] = friendId
            map = RsaUtils.encryptedData("POST", CHECK_RELATION_STATUS, map)

            if (BaseRequestApi.iRetrofitModel == null) {
                BaseRequestApi.iRetrofitModel = CreateRetrofit.createService(IRetrofitModel::class.java)
            }

            val call = BaseRequestApi.iRetrofitModel.checkRelationStatus(map)
            call.enqueue(callback)
        }
        
        
        
         /**
         * 检查剩余发送次数
         */
        fun checkSendMessageNum(accessToken: String, groupId: String, callback: ApiResponseCallback<BaseRetrofitModel<Any>>) {
            var map: MutableMap<String, Any> = CommonUtils.createMap()

            map[ACCESS_TOKEN] = accessToken

            map = RsaUtils.encryptedData("POST", "$CHECK_SEND_MESSAGE/$groupId", map)

            if (BaseRequestApi.iRetrofitModel == null) {
                BaseRequestApi.iRetrofitModel = CreateRetrofit.createService(IRetrofitModel::class.java)
            }

            val call = BaseRequestApi.iRetrofitModel.checkSendMessageNum(map, groupId)
            call.enqueue(callback)
        }
        
        
        org.gradle.api.tasks.TaskExecutionException: Execution failed for task ':app:compileDebugKotlin'.
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:84)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.execute(ExecuteActionsTaskExecuter.java:55)
	at org.gradle.api.internal.tasks.execution.SkipUpToDateTaskExecuter.execute(SkipUpToDateTaskExecuter.java:62)
	at org.gradle.api.internal.tasks.execution.ValidatingTaskExecuter.execute(ValidatingTaskExecuter.java:58)
	at org.gradle.api.internal.tasks.execution.SkipEmptySourceFilesTaskExecuter.execute(SkipEmptySourceFilesTaskExecuter.java:88)
	at org.gradle.api.internal.tasks.execution.ResolveTaskArtifactStateTaskExecuter.execute(ResolveTaskArtifactStateTaskExecuter.java:46)
	at org.gradle.api.internal.tasks.execution.SkipTaskWithNoActionsExecuter.execute(SkipTaskWithNoActionsExecuter.java:51)
	at org.gradle.api.internal.tasks.execution.SkipOnlyIfTaskExecuter.execute(SkipOnlyIfTaskExecuter.java:54)
	at org.gradle.api.internal.tasks.execution.ExecuteAtMostOnceTaskExecuter.execute(ExecuteAtMostOnceTaskExecuter.java:43)
	at org.gradle.api.internal.tasks.execution.CatchExceptionTaskExecuter.execute(CatchExceptionTaskExecuter.java:34)
	at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter$EventFiringTaskWorker$1.execute(DefaultTaskGraphExecuter.java:236)
	at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter$EventFiringTaskWorker$1.execute(DefaultTaskGraphExecuter.java:228)
	at org.gradle.internal.Transformers$4.transform(Transformers.java:169)
	at org.gradle.internal.progress.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:106)
	at org.gradle.internal.progress.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:61)
	at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter$EventFiringTaskWorker.execute(DefaultTaskGraphExecuter.java:228)
	at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter$EventFiringTaskWorker.execute(DefaultTaskGraphExecuter.java:215)
	at org.gradle.execution.taskgraph.AbstractTaskPlanExecutor$TaskExecutorWorker.processTask(AbstractTaskPlanExecutor.java:77)
	at org.gradle.execution.taskgraph.AbstractTaskPlanExecutor$TaskExecutorWorker.run(AbstractTaskPlanExecutor.java:58)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:54)
	at org.gradle.internal.concurrent.StoppableExecutorImpl$1.run(StoppableExecutorImpl.java:40)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: org.gradle.api.GradleException: Compilation error. See log for more details
	at org.jetbrains.kotlin.gradle.tasks.TasksUtilsKt.throwGradleExceptionIfError(tasksUtils.kt:16)
	at org.jetbrains.kotlin.gradle.tasks.KotlinCompile.processCompilerExitCode(Tasks.kt:411)
	at org.jetbrains.kotlin.gradle.tasks.KotlinCompile.callCompiler$kotlin_gradle_plugin(Tasks.kt:385)
	at org.jetbrains.kotlin.gradle.tasks.KotlinCompile.callCompiler$kotlin_gradle_plugin(Tasks.kt:253)
	at org.jetbrains.kotlin.gradle.tasks.AbstractKotlinCompile.execute(Tasks.kt:215)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.gradle.internal.reflect.JavaMethod.invoke(JavaMethod.java:73)
	at org.gradle.api.internal.project.taskfactory.DefaultTaskClassInfoStore$IncrementalTaskAction.doExecute(DefaultTaskClassInfoStore.java:163)
	at org.gradle.api.internal.project.taskfactory.DefaultTaskClassInfoStore$StandardTaskAction.execute(DefaultTaskClassInfoStore.java:134)
	at org.gradle.api.internal.project.taskfactory.DefaultTaskClassInfoStore$StandardTaskAction.execute(DefaultTaskClassInfoStore.java:123)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeAction(ExecuteActionsTaskExecuter.java:95)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:76)
	... 23 more