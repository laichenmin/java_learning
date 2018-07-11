##kafka安装
####step 1:下载kafka  
<pre>
1. > wget http://mirror.bit.edu.cn/apache/kafka/1.1.0/kafka_2.11-1.1.0.tgz

</pre>

官网下载地址：http://kafka.apache.org/quickstart
####step 2:解压并安装kafka####
<pre>
1. > tar -xzf kafka_2.11-1.1.0.tgz
2. > cd kafka_2.11-1.1.0
</pre>

####step 3:修改kafka配置文件####
<pre>
1. > vi config/server.properties
</pre>

    listeners=PLAINTEXT://:9092
    #远程服务器IP地址
    advertised.listeners=PLAINTEXT://47.104.158.74:9092
    #zookeeper地址和端口
    zookeeper.connect=47.104.158.74:2181

####step 4:启动 zookeeper 和 kafka####
启动zookeeper
<pre>
1. > bin/zookeeper-server-start.sh config/zookeeper.properties &  
</pre>
启动kafka  
<pre>
1. > bin/kafka-server-start.sh config/server.properties &
</pre>
    
##spring boot集成kafka##
####step 1:pom文件引入####
    
	<dependency>
	    <groupId>org.springframework.kafka</groupId>
	    <artifactId>spring-kafka</artifactId>
	</dependency>
    
####step 2:配置文件####
    kafka.consumer.servers=47.104.158.74:9092
    kafka.consumer.enable.auto.commit=true
    kafka.consumer.session.timeout=6000
    kafka.consumer.auto.commit.interval=100
    kafka.consumer.auto.offset.reset=latest
    kafka.consumer.group.id=kafka-test-group
    kafka.consumer.concurrency=10

    kafka.producer.servers=47.104.158.74:9092
    kafka.producer.retries=1
    kafka.producer.batch.size=4096
    kafka.producer.linger=1
    kafka.producer.buffer.memory=40960
    
####step 3:生产者配置类####
	@Configuration
	@EnableKafka
	public class KafkaProducerConfig {
	    @Value("${kafka.producer.servers}")
	    private String servers;
	    @Value("${kafka.producer.retries}")
	    private int retries;
	    @Value("${kafka.producer.batch.size}")
	    private int batchSize;
	    @Value("${kafka.producer.linger}")
	    private int linger;
	    @Value("${kafka.producer.buffer.memory}")
	    private int bufferMemory;
	    @Bean
	    public KafkaTemplate<String, String> kafkaTemplate() {
	        return new KafkaTemplate(producerFactory());
	    }
	    public ProducerFactory<String, String> producerFactory() {
	        return new DefaultKafkaProducerFactory<>(producerConfigs());
	    }
	    public Map<String, Object> producerConfigs() {
	        Map<String, Object> props = new HashMap<>();
	        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
	        props.put(ProducerConfig.RETRIES_CONFIG, retries);
	        props.put(ProducerConfig.BATCH_SIZE_CONFIG, batchSize);
	        props.put(ProducerConfig.LINGER_MS_CONFIG, linger);
	        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, bufferMemory);
	        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
	        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
	        return props;
	    }
	}

####step 4:消费者配置类####
	@Configuration
	@EnableKafka
	public class KafkaConsumerConfig {
	    @Value("${kafka.consumer.servers}")
	    private String servers;
	    @Value("${kafka.consumer.enable.auto.commit}")
	    private boolean enableAutoCommit;
	    @Value("${kafka.consumer.session.timeout}")
	    private String sessionTimeout;
	    @Value("${kafka.consumer.auto.commit.interval}")
	    private String autoCommitInterval;
	    @Value("${kafka.consumer.group.id}")
	    private String groupId;
	    @Value("${kafka.consumer.auto.offset.reset}")
	    private String autoOffsetReset;
	    @Value("${kafka.consumer.concurrency}")
	    private int concurrency;
	    @Bean
	    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
	        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
	        factory.setConsumerFactory(consumerFactory());
	        factory.setConcurrency(concurrency);
	        factory.getContainerProperties().setPollTimeout(1500);
	        return factory;
	    }
	    public ConsumerFactory<String, String> consumerFactory() {
	        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
	    }
	    public Map<String, Object> consumerConfigs() {
	        Map<String, Object> propsMap = new HashMap<>(8);
	        propsMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
	        propsMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, enableAutoCommit);
	        propsMap.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, autoCommitInterval);
	        propsMap.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, sessionTimeout);
	        propsMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
	        propsMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
	        propsMap.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
	        propsMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, autoOffsetReset);
	        return propsMap;
	    }
	}
####step 5:生产者类####
	@Component
	public class KafkaProducer {
	    private Logger logger = LoggerFactory.getLogger(getClass());
	    @Autowired
	    private KafkaTemplate kafkaTemplate;
	
	    public void sendMessage(String topic, String message) {
	        logger.info("send message======="+message);
	        kafkaTemplate.send(topic, message);
	    }
	}
####step 6:消费者类####
	@Component
	public class VideoCosConsumer {
	    protected final Logger logger = LoggerFactory.getLogger(this.getClass());
	    private Gson gson = new GsonBuilder().create();
	
	    @KafkaListener(topics = {"test-topic"})
	    public void consumerMessage(String message) {
	        UserEntity m = gson.fromJson(message, UserEntity.class);
	        logger.info("receive message======="+message);
	    }
	
	}

参考地址：  
http://kafka.apache.org/quickstart  

