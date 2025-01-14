# RODPS: ODPS Plugin for R

[![Building RODPS](https://github.com/aliyun/aliyun-odps-r-plugin/actions/workflows/building.yaml/badge.svg?branch=master)](https://github.com/aliyun/aliyun-odps-r-plugin/actions/workflows/building.yaml)


## Features

- Read/write dataframe from/to ODPS.
- Convert some of the R models to SQL command.
- The large data set can be processed by using the distributed algorithm.
- The small data set can be processed directly in R.

## Requirements

System dependencies:

- Java 8+
- R 1.8+

R libraries:

- [rJava](https://cran.r-project.org/web/packages/rJava/index.html)
- [DBI](https://cran.r-project.org/web/packages/DBI/index.html)
- [RSQLite](https://cran.r-project.org/web/packages/RSQLite/index.html)

## Installation

1. Install the R dependencies:

```R
install.packages('DBI')
install.packages('rJava')
install.packages('RSQLite')
```
Or you can use `devtools` to help you resolve dependencies:

```R
install.packages('devtools')
```

2. Install RODPS

    2.1. Install from release package
   
    ```R
    install.packages('https://github.com/aliyun/aliyun-odps-r-plugin/releases/download/v2.1.3/RODPS_2.1.3.tar.gz', type="source", repos=NULL)
    ```

    2.2. Install from source

    * Clone the source repo and build with `./build.sh`
    * Install built package: `R CMD INSTALL target/RODPS_2.1.3.tar.gz`



## Getting Started

1. Please make sure the environment variable `RODPS_CONFIG` is set to `path/to/odps_config.ini`

```bash
export RODPS_CONFIG=path/to/odps_config.ini
```

See the configuration template: [odps_config.ini.template](https://github.com/aliyun/aliyun-odps-r-plugin/blob/master/odps_config.ini.template)

2. Basic RODPS functions:

```R
> library("RODPS")  # Load RODPS
>
> tbl1 <- rodps.table.read("tbl1")  # read dataframe from ODPS
> d <- head(tbl1)
>
> rodps.sql('create table test_table(id bigint);')   # execute sql
>
> names(iris) <- gsub("\\.","_",names(iris))                                   # rename columns
> rodps.table.write(iris, 'iris')                                              # write dataframe to ODPS
>
> rodps.table.sample.srs('tbl1','small_tbl1', 100 )                            # sampling by raw
>
> rodps.table.hist(tblname='iris', colname='species', col=rainbow(10), freq=F) # create a histogram
>
> library(rpart)
> fit <- rpart(Species~., data=iris)
> rodps.predict.rpart(fit, srctbl='iris',tgttbl='iris_p')                      # modeling
```

## Under the Hood

### Design Architecture

For the mind map of related concepts, please refer to the [MindMapDoc](https://github.com/aliyun/aliyun-odps-r-plugin/blob/master/mindmap.pdf)

### Type System

**All numeric in R have possibility of precision loss.**

| MaxCompute/ODPS | R | Notes |
|-----------------|---|-------|
| BOOLEAN | logical | |
| BIGINT | numeric | \[-9223372036854774784, 9223372036854774784\] * |
| INT | numeric | |
| TINYINT | numeric | |
| SMALLINT | numeric | |
| DOUBLE | numeric | |
| FLOAT | numeric | |
| DATETIME | numeric | POSIXct POSIXlt, in second |
| DATE | numeric | POSIXct POSIXlt, in second |
| TIMESTAMP | numeric | POSIXct POSIXlt, in second |
| INTERVAL_YEAR_MONTH | numeric | in month |
| INTERVAL_DATE_TIME | numeric | in second |
| DECIMAL | numeric | |
| STRING | character | |
| CHAR | character | |
| VARCHAR | character | |
| BINARY | character | |
| MAP | - | unsupport |
| ARRAY | - | unsupport |
| STRUCT | - | unsupport |

* BIGINT(64bit) from MaxCompute is stored and calculated as double(64bit) in RODPS. Precision loss might happen when casting BIGINT to double, which shrinks the min/max value could be written back to MaxCompute/ODPS.

### Trouble shooting

- For Windows users: DO NOT install BOTH 32bit and 64bit R on your system, which will introduce compiling issues in the installation of `rJava`.

## License

Licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0.html)
