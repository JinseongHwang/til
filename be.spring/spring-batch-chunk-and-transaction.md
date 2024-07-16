## Spring Batch의 실행 계층

- Job > Step > Tasklet
- Tasklet은 Reader + Processor + Writer 로 구성할 수도 있다.

## JobBuilderFactory, StepBuilderFactory

```kotlin
@Configuration
class FooJobConfiguration(
    private val jobBuilderFactory: JobBuilderFactory,
    private val stepBuilderFactory: StepBuilderFactory,
) {
    // ...
}
```
- 보통 Job을 만들 때면 위와 같은 코드에서 생성하게 된다.
- 습관처럼 JobBuilderFactory, StepBuilderFactory를 빈 주입 받는다. 이 빈들은 `@EnableBatchProcessing`를 통해 배치 활성화 시 빈으로 등록된다.
  - 버전에 따라 어디서 등록을 시켜주는지는 조금씩 바뀌는 것 같다.
  - 4버전은 SimpleBatchConfiguration (이 글의 내용은 4버전)

---

## Chunk Oriented는 어떻게 동작할까? 실제 코드를 살펴보자

```java
public class SimpleChunkProvider<I> implements ChunkProvider<I> {

	protected final ItemReader<? extends I> itemReader;

	public SimpleChunkProvider(ItemReader<? extends I> itemReader, RepeatOperations repeatOperations) {
		this.itemReader = itemReader; // ==> 생성자에서 itemReader를 받아서
		this.repeatOperations = repeatOperations;
	}
	
	@Nullable
	protected final I doRead() throws Exception {
		try {
			listener.beforeRead();
			I item = itemReader.read(); // ==> 더 이상 없을 때까지 읽는다
			if(item != null) {
				listener.afterRead(item);
			}
			return item;
		}
		// ...
	}
	
	@Override
	public Chunk<I> provide(final StepContribution contribution) throws Exception {
		final Chunk<I> inputs = new Chunk<>();
		repeatOperations.iterate(new RepeatCallback() {

			@Override
			public RepeatStatus doInIteration(final RepeatContext context) throws Exception {
				I item = null;
				try {
					item = read(contribution, inputs);
				}
				if (item == null) {
					inputs.setEnd();
					return RepeatStatus.FINISHED;
				}
				inputs.add(item); // ==> 읽은 item 1개씩 chunk에 넣어준다
			}

		});
        // ...
		return inputs; // ==> chunk를 반환한다
	}
	// ...
}
```
- SimpleChunkProvider는 ItemReader를 가진다.
- ItemReader로 chunk를 가득 채울 때까지 읽는다.
- 그리고 chunk를 반환한다

```java
public class SimpleChunkProcessor<I, O> implements ChunkProcessor<I>, InitializingBean {

	private ItemProcessor<? super I, ? extends O> itemProcessor;
	private ItemWriter<? super O> itemWriter;

	public SimpleChunkProcessor(@Nullable ItemProcessor<? super I, ? extends O> itemProcessor, ItemWriter<? super O> itemWriter) {
		this.itemProcessor = itemProcessor; // ==> Processor를 가진다. nullable이다.
		this.itemWriter = itemWriter; // ==> Writer를 가진다. 필수이다.
	}
	
	protected final O doProcess(I item) throws Exception {
		if (itemProcessor == null) {
			@SuppressWarnings("unchecked")
			O result = (O) item;
			return result; // ==> 프로세서가 없다면 doProcess 종료
		}

		try {
			listener.beforeProcess(item);
			O result = itemProcessor.process(item); // ==> 프로세서가 있다면 아이템 1개씩 process
			listener.afterProcess(item, result);
			return result;
		}
		catch (Exception e) {
			listener.onProcessError(item, e);
			throw e;
		}
        // ...
	}

	protected final void doWrite(List<O> items) throws Exception { // ==> 매개변수로 item list를 받는다.
		try {
			listener.beforeWrite(items);
			writeItems(items); // ==> items를 한꺼번에 write한다.
			doAfterWrite(items);
		}
		catch (Exception e) {
			doOnWriteError(e, items);
			throw e;
		}
		// ...
	}
	// ...
}
```
- SimpleChunkProcessor는 ItemProcessor, ItemWriter를 가진다.
  - 다만 ItemProcessor는 필수가 아니다.
- doProcess
  - 프로세서가 있다면 아이템 1건씩 process 한다
  - 프로세서가 없다면 받으 데이터를 그대로 다시 return 한다
- doWrite
  - 매개변수로 List<I> (item list)를 받는다.
  - item list를 한방에 write 한다.

## Tasklet 만들기

```java
public class ChunkOrientedTasklet<I> implements Tasklet {

	private final ChunkProcessor<I> chunkProcessor;
	private final ChunkProvider<I> chunkProvider;

	public ChunkOrientedTasklet(ChunkProvider<I> chunkProvider, ChunkProcessor<I> chunkProcessor) {
		this.chunkProvider = chunkProvider;
		this.chunkProcessor = chunkProcessor;
	}

	@Nullable
	@Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
		@SuppressWarnings("unchecked")
		Chunk<I> inputs = (Chunk<I>) chunkContext.getAttribute(INPUTS_KEY);
		if (inputs == null) {
			inputs = chunkProvider.provide(contribution);
			if (buffering) {
				chunkContext.setAttribute(INPUTS_KEY, inputs);
			}
		}

		chunkProcessor.process(contribution, inputs);
		chunkProvider.postProcess(contribution, inputs);
		// ...
	}
	// ...
}
```
- ChunkOrientedTasklet는 앞서 봤던 ChunkProvider, ChunkProcessor를 가진다.
- ChunkProvider의 로직, ChunkProcessor의 로직을 차례로 실행한다.
- StepBuilderFactory에 reader, processor, writer를 등록하고 마지막에 build를 호출하면 TaskletStep을 반환하는 것도 이 이유이다.
```java
// SimpleStepBuilder.java
@Override
protected Tasklet createTasklet() {
    Assert.state(reader != null, "ItemReader must be provided");
    Assert.state(writer != null, "ItemWriter must be provided");
    RepeatOperations repeatOperations = createChunkOperations();
    SimpleChunkProvider<I> chunkProvider = new SimpleChunkProvider<>(getReader(), repeatOperations);
    SimpleChunkProcessor<I, O> chunkProcessor = new SimpleChunkProcessor<>(getProcessor(), getWriter());
    chunkProvider.setListeners(new ArrayList<>(itemListeners));
    chunkProcessor.setListeners(new ArrayList<>(itemListeners));
    ChunkOrientedTasklet<I> tasklet = new ChunkOrientedTasklet<>(chunkProvider, chunkProcessor);
    tasklet.setBuffering(!readerTransactionalQueue);
    return tasklet;
} 
```

## Chunk Oriented 동작 정리

100건의 데이터, chunk_size=10 이라고 가정했을 때,

1. 1개씩 read를 10번해서 chunk를 1개 만든다.
2. chunk 내 item 1개씩 process를 10번한다.
3. processed chunk (10건) 를 write 한다.

---

## 그럼 트랜잭션은 ?

- StepBuilderFactory의 동작을 쭉 따라가다 보면 SimpleStepBuilder 에서 build() 를 발견할 수 있다. build()는 TaskletStep 를 반환한다.
- TaskletStep의 doExecute() 를 살펴보면 다음과 같은 코드가 나온다.

```java
result = new TransactionTemplate(transactionManager, transactionAttribute)
    .execute(new ChunkTransactionCallback(chunkContext, semaphore));
```

- StepBuilderFactory에서 인자로 받던 TransactionManager의 행방을 찾았다!
- 결론적으로 개별 Step이 가지고 있는 TransactionManager에 의해 Chunk Tasklet이 실행된다는 것을 알 수 있다.

## 트랜잭션 정리

- Step이 1번 실행될 때마다 트랜잭션이 1번 실행된다.
- Step 안에는 reader, processor, writer가 동작하므로 어디선가 쿼리 실패하면 전체 롤백이 가능한 것이다.

## 유의사항

- 다만 Step 전체 과정에서 트랜잭션 1개를 계속 점유하고 있으니, Step size를 적당하게 잡는 것이 중요하다.
  - (여기서 말하는 Step size는 소요 시간)
- Step size가 너무 크면 Job의 실행 속도는 빨라지겠지만 DBCP 리소스가 금방 바닥날 수도 있다.
- Step size가 너무 작으면 Job의 실행 속도는 느려지겠지만 DBCP 리소스를 사이 좋게 나눠 쓸 수 있다.

---

## References

- https://www.sktenterprise.com/bizInsight/blogDetail/dev/2566