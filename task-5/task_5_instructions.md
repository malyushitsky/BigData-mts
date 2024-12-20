# Гайд по выполнению 5 задания

## Скачиваем и настраиваем prefect

### 1. Подключение к Name Node
   ```bash
   ssh team-19-nn
   ```
### 2. Зайдем в юзера hadoop
   ```bash
   sudo -i -u hadoop
   ``` 
### 3. Активируем виртуальное окружение, созданное в домашнем задании 4 source 
   ```bash
   source .venv/bin/activate
   ``` 
### 4. Установим prefect
   ```bash
   pip install prefect
   ``` 
### 5. Создадим файл с функциями для prefect через nano (nano prefect_flow)
   ```python
from prefect import flow, task
from pyspark.sql import DataFrame, SparkSession
from pyspark.sql import functions as F

@task(name="Create Spark Session")
def create_spark_session(app_name: str = "spark") -> SparkSession:
    spark = (
        SparkSession.builder.master("yarn") \
        .appName("spark-with-yarn") \
        .config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
        .config("spark.hadoop.hive.metastore.uris", "thrift://ip_namenode:5433") \
        .enableHiveSupport() \
        .getOrCreate()
    )

    return spark


@task(name="Read CSV")
def read_csv(
    spark: SparkSession,
    file_path: str = "/input/organizations-2000000.csv",
) -> DataFrame:
    df = spark.read.csv(file_path,
                        header=True,
                        inferSchema=True)  
    return df


@task(name="Calculate aggregates")
def calc_aggs(df: DataFrame) -> DataFrame:

   transformed_df = df.select([
   F.max(df.Founded),
   F.min(df.Index),
   F.mean(df['Number of employees']),
   F.count_distinct(df.Country),
   F.count_distinct(df.Name)])

   return transformed_df


@task(name="Repartition df")
def repartition_df(
    df: DataFrame, 
    n_partitions: int = 2,
    column: str = "Founded"
) -> DataFrame:

    df = df.repartition(n_partitions, column)

    return df

@task(name="Save parquet into hdfs")
def save_parquet(df: DataFrame,
                 output_path: str = "/out/organizations"):

    df.write.parquet(output_path)


@task(name="Save table into Hive")
def save_hive(
    df: DataFrame,
    table: str = "test_df",
    partition_column: str = "Founded",
):
    df.write.saveAsTable(table_name, partitionBy=partition_column)


@flow(name="Test flow")
def test_flow():
    spark = create_spark_session()

    df = read_csv(spark)
    transformed_df = calc_aggs(df)
    df = repartition_df(df)

    save_parquet(df)
    save_as_hive_table(df)

if __name__ == "__main__":
    test_flow()
   ``` 
### 6. Запустим prefect_flow 
   ```bash
   python3 prefect_flow
   ``` 

### 5. Создадим файл с функциями для запуска prefect по расписанию (через nano prefect_flow_scheduled)
   ```python
from prefect import flow, task
from pyspark.sql import DataFrame, SparkSession
from pyspark.sql import functions as F

@task(name="Create Spark Session")
def create_spark_session(app_name: str = "spark") -> SparkSession:
    spark = (
        SparkSession.builder.master("yarn") \
        .appName("spark-with-yarn") \
        .config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
        .config("spark.hadoop.hive.metastore.uris", "thrift://ip_namenode:5433") \
        .enableHiveSupport() \
        .getOrCreate()
    )

    return spark


@task(name="Read CSV")
def read_csv(
    spark: SparkSession,
    file_path: str = "/input/organizations-2000000.csv",
) -> DataFrame:
    df = spark.read.csv(file_path,
                        header=True,
                        inferSchema=True)  
    return df


@task(name="Calculate aggregates")
def calc_aggs(df: DataFrame) -> DataFrame:

   transformed_df = df.select([
   F.max(df.Founded),
   F.min(df.Index),
   F.mean(df['Number of employees']),
   F.count_distinct(df.Country),
   F.count_distinct(df.Name)])

   return transformed_df


@task(name="Repartition df")
def repartition_df(
    df: DataFrame, 
    n_partitions: int = 2,
    column: str = "Founded"
) -> DataFrame:

    df = df.repartition(n_partitions, column)

    return df

@task(name="Save parquet into hdfs")
def save_parquet(df: DataFrame,
                 output_path: str = "/out/organizations_prefect"):

    df.write.parquet(output_path)


@task(name="Save table into Hive")
def save_hive(
    df: DataFrame,
    table: str = "table_prefect",
    partition_column: str = "Founded",
):
    df.write.saveAsTable(table_name, partitionBy=partition_column)


@flow(name="Test flow")
def test_flow():
    spark = create_spark_session()

    df = read_csv(spark)
    transformed_df = calc_aggs(df)
    df = repartition_df(df)

    save_parquet(df)
    save_hive(df)
   ```

### 6. Установим корректный url для prefect 
   ```bash
   export PREFECT_URL="http://team-19-nn:port/api"
   ``` 
### 7. Запустим сервер prefect
   ```bash
   prefect server start --host team-19-nn --port port -b
   ``` 
### 8. Установим расписание для prefect flow
   ```bash
   prefect flow serve prefect_flow_scheduled:test_flow --cron "0 * * * *" --name "hour" --no-pause-on-shutdown
   ``` 





