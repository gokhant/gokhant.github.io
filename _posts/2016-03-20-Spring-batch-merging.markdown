---
layout: post
title:  "48-Spring Batch; merging lines"
date:   2016-03-20 20:42:05 +0200
excerpt: "Spring batch continues, how to merge the lines that have related information"
categories: 
- programming
- java
- springframework
- batch

tags:
- spring
- batch
- java
- csv
- gsm
- telecommunication
- cell
- basestation
- timing advance
- propagation delay

---

In my [post 50]({% post_url 2016-02-28-Spring-batch %}), I stated that, there are 24 lines of information for each cell; 
mainly number of subscribers at each hour according to their distances to the cell.


- Each line in the csv file is a ***Measurement***. 

~~~~ java
. . .
public class Measurement {
    private String cellId;
    private String dataTime;
    private List<Counter> counters = new ArrayList<Counter>();
. . .
~~~~

- Every value in a line is a ***Counter***.

~~~~ java
. . .
public class Counter implements Comparable<Counter> {
    private String name;
    private int value;
. . .
~~~~

- ***Result*** is the class my application produces after processing data of each cell.

~~~~ java
. . .
public class Result {
    String cellId;
    Counter counter;
. . .
~~~~

My first implementation of ItemProcessor has forced me to process the lines from the csv file one by one. 
Using a simple *previous* variable was there for the rescue which enabled the application to collect the 
records related to each cell easily. As you can see from the code below, calculations are done when a new cell arrives.

\\
There is **one bug** (maybe more) and **a limitation** in this approach as far as I experienced. Can you see them?

~~~~ java
. . .
public class Processor implements ItemProcessor<Measurement, Result> {
    Logger logger = LoggerFactory.getLogger(Processor.class);
    Measurement previous;

    @Autowired
    CacheServiceDef cacheService;

    @Override
    public Result process(Measurement measurement) throws Exception {
        Result result = null;
        logger.debug("[process] processing " + measurement);
        if (previous == null) {
            previous = measurement;
        } else {
            if (measurement.isSame(previous)) {
                previous.merge(measurement);
            } else {
                result = calculate(previous);
                previous = measurement;
            }
        }
        return result;
    }

    private Result calculate(Measurement previous) {
        Result result = null;
        List<Counter> counters = previous.getCounters();
        Counter max = Collections.max(counters);
        logger.debug("Max counter is " + max);
        Cell cell = cacheService.getCell(previous.getCellId());
        if (cell == null) {
            logger.warn("Cell " + previous.getCellId() + " does not exist in cache");
        } else {
            result = new Result();
            result.setCounter(max);
            result.setCell(cell);
        }
        return result;
    }
}
~~~~

But before that, there is also a service to cache all cell information and then query it whenever necessary. At work, this information is cached from a database; 
but due to confidentiality issues, I can not share it here. That's why, I filled the cache with a helper class **CellGeneratorUtil**. 
It simply creates given number of cells with ordered cellIds, random azimuth, beamwidth, latitude and longitudes and let the **CacheService** cache them.

~~~~ java
@Service
public class SampleCacheService implements CacheServiceDef{
    Logger logger = LoggerFactory.getLogger(SampleCacheService.class);
    Map<Integer, Cell> cells = null;
    @Override
    public Cell getCell(int cellId) {
        return cells.get(cellId);
    }

    @PostConstruct
    public void initialize() {
        cells = new CellGeneratorUtil().createCells(20000);
        logger.debug(cells.size() + " cells have been cached");
    }
}
. . .
~~~~


Code below lets us to write our results to a given file, what if it's required 

- to write to more than one file

- to append header and/or footer to some of the files?

~~~~ java
@Configuration
@EnableBatchProcessing
public class BatchConfiguration {
    Logger logger = LoggerFactory.getLogger(BatchConfiguration.class);
		. . .  
    @Bean
    public ItemWriter<Result> resultWriter() {
        return resultWriter(
                new FileSystemResource("results.csv")
        );
    }
    . . .
    private ItemWriter<Result> resultWriter(FileSystemResource fsr) {
        FlatFileItemWriter<Result> writer = new FlatFileItemWriter<Result>();
        writer.setResource(fsr);
        writer.setShouldDeleteIfExists(true);
        DelimitedLineAggregator<Result> lineAggregator = new DelimitedLineAggregator<Result>();
        lineAggregator.setDelimiter(",");
        BeanWrapperFieldExtractor<Result> fExtractor = new BeanWrapperFieldExtractor<Result>();
        fExtractor.setNames(new String[] {"cell.cellId", "cell.latitude", "cell.longitude", "cell.azimuth", "cell.beamWidth", "counter.name", "counter.value"});
        lineAggregator.setFieldExtractor(fExtractor);
        writer.setLineAggregator(lineAggregator);
        return writer;
    }
}
~~~~


Let's return back to the drawback and limitation I have mentioned minutes ago. 

- Limitation first; 24 measurements of cell should be altogether, there should be no other lines among them. 
What is required, if this is the case; measurements of cells are mixed in a file?

- The bug, is that, the application tracks the changes of Cell in the file and then it processes the previous data; 
what about the measurements of the last cell in the file? There will be no other cell after it, 
so the measurements of the last cell will never be calculated. Can you tink of some ways to overcome this issue?

Stay tuned!