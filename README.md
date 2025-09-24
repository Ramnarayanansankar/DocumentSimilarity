

---

# Assignment 2: Document Similarity using MapReduce

**Name:** Ramnarayanan Sankar
**Student ID:** 801409708

---

## Approach and Implementation

### Mapper Design

The `DocumentSimilarityMapper` class processes each document by taking a line of text as input. For each document, it extracts the filename to use as a unique ID. It then tokenizes the document's content, converts each word to lowercase, and stores them in a HashSet to keep only the unique words. Finally, it emits a key-value pair where the key is the document's filename and the value is a single, comma-separated string of all its unique words.

This process efficiently transforms raw text into a structured format, preparing the data for the Reducer to easily compare word sets and calculate the **Jaccard Similarity** between document pairs.

### Reducer Design

The `DocumentSimilarityReducer` class calculates the Jaccard Similarity for every possible pair of documents. It receives a document's filename and its unique word set from the mapper. The reducer maintains a map to store the word sets of all documents it has processed.

For each new document it receives:

1. It compares it against every document already in the map.
2. For each pair, it computes the **intersection** (common words) and **union** (total unique words) of their word sets.
3. The Jaccard Similarity is then calculated as:

$$
\text{Jaccard Similarity} = \frac{|Intersection|}{|Union|}
$$

Finally, it emits the pair of document names and their corresponding similarity score.

### Overall Data Flow

1. **Input files** are placed inside `shared_folder/input_files`.
2. **Mapper Phase:** Each document is tokenized into unique words, producing `(filename, wordSet)` pairs.
3. **Shuffle & Sort Phase:** The framework groups intermediate outputs by keys (document identifiers).
4. **Reducer Phase:** Each new document is compared with all previously processed ones. For each document pair, Jaccard similarity is computed.
5. **Output:** Results are written to the specified HDFS output folder and then can be copied back to the local machine.

---

## Setup and Execution

> ⚠️ *Note: The following commands were tested in Codespaces with Hadoop running in Docker. Adjust paths if needed.*

### 1. Start the Hadoop Cluster

```bash
docker compose up -d
```

### 2. Build the Code

```bash
mvn clean install
```

### 3. Copy the JAR to Docker Container

```bash
docker cp target/DocumentSimilarity-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 4. Move Dataset to Docker Container

```bash
docker cp shared_folder/input_files/ resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 5. Connect to Docker Container

```bash
docker exec -it resourcemanager /bin/bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 6. Set Up HDFS

```bash
hadoop fs -mkdir -p /input/dataset
hadoop fs -put ./input_files /input/dataset
```

### 7. Execute the MapReduce Job

```bash
hadoop jar DocumentSimilarity-0.0.1-SNAPSHOT.jar com.example.controller.DocumentSimilarityDriver /input/dataset/input_files /output
```

(*If `/output` already exists, use `/output1` or another name.*)

### 8. View the Output

```bash
hadoop fs -cat /output/*
```

### 9. Copy Output to Local Machine

Inside the container:

```bash
hdfs dfs -get /output /opt/hadoop-3.2.1/share/hadoop/mapreduce/
exit
```

From local terminal:

```bash
docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output/ shared_folder/output/
```

---

## Challenges and Solutions

1. **Duplicate output folder error:** Hadoop does not allow writing to an existing output directory.

   * ✅ Solution: Changed the output folder name (e.g., `/output1`).

2. **Pairwise comparison complexity:** Comparing every document with every other document can be computationally heavy.

   * ✅ Solution: Used a `HashSet` to quickly compute intersections and unions.

3. **Environment setup (Hadoop in Docker):** Initial difficulty in configuring Hadoop + ensuring dataset is accessible.

   * ✅ Solution: Used Docker + HDFS setup via provided `docker-compose.yml`, copied datasets and JAR into container.

---

## Sample Input

**Input (`small_dataset.txt`)**

```
Document1 This is a sample document containing words
Document2 Another document that also has words
Document3 Sample text with different words
```

## Sample Output

**Expected Output**

```
"Document1, Document2 Similarity: 0.56"
"Document1, Document3 Similarity: 0.42"
"Document2, Document3 Similarity: 0.50"
```

**Obtained Output:**
(Place your actual Hadoop output here once you run the job.)

---

## GitHub & Workflow

* **Repository Structure:**

  * `src/main/java/com/example` → Mapper, Reducer, Driver classes
  * `shared_folder/input_files` → Input datasets
  * `shared_folder/output` → Results from HDFS
  * `docker-compose.yml` & `hadoop.env` → Hadoop cluster setup
  * `README.md` → Documentation

* **Tools Used:**

  * GitHub (commits, repo structure, version control)
  * GitHub Codespaces (for containerized development environment)
  * Hadoop in Docker (distributed execution)
  * Maven (build system)

---

## Performance Comparison

| Configuration   | Execution Time | Notes                            |
| --------------- | -------------- | -------------------------------- |
| **3 Datanodes** | 13 sec         | Parallel execution across nodes  |
| **1 Datanode**  | 24 sec         | All tasks handled by single node |


---