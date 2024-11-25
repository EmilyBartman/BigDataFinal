Emily Bartman

11/23/2024

Fall 2024

Course Project: Big Complex Data

#Introduction
Anime, as a globally popular entertainment medium, has captivated audiences with its diverse storytelling and vibrant characters. However, with the growing volume of anime titles, navigating and analyzing trends within the industry poses significant challenges. This project leverages big data technologies to analyze a comprehensive dataset of anime, focusing on user preferences, ratings trends, and metadata such as genres, studio influence, and popularity over time. The analysis provides actionable insights for fans, producers, and distributors, offering tools to better understand the factors driving anime success and audience engagement.


#Background
The topic of anime analytics was chosen because of my personal interest in the genre and its importance as a cultural phenomenon. The global appeal of anime has led to a diverse range of titles, making it challenging for viewers to discover content tailored to their tastes. Moreover, the industry lacks a centralized platform for robust data-driven decision-making, leaving room for innovation. By addressing this topic, the project highlights the importance of applying big data techniques to solve real-world problems in niche industries, offering potential tools for anime producers and platforms to optimize content delivery and marketing strategies.

#Methodology
To address the problem, I used publicly available datasets containing detailed metadata on anime titles, user ratings, and popularity metrics. The methodology involved the following:

### **Technological Setup**
The project utilized the following technologies:
- **Google Cloud Platform (GCP):** For storage and processing of the anime dataset.
- **Pandas:** For data manipulation and cleaning.
- **Matplotlib:** For data visualization.
- **PyMongo and MongoDB:** For potential integration of NoSQL database functionality.
- **Kaggle API:** For accessing external datasets.
- **GitHub:** For publishing code and results.

### **Steps Taken**
1. **Data Retrieval from GCS:** The dataset was retrieved from a GCS bucket (`my_anime_list`) and loaded into a Pandas DataFrame.
2. **Data Cleaning:** Missing values in critical columns (`genre`, `score`) were handled, and non-numeric values were coerced to maintain consistency.
3. **Analysis:** The dataset was grouped by `genre` to calculate the average score, identifying top genres by popularity.
4. **Visualization:** A bar chart illustrated the top genres by average score.
5. **Augmentation:** Additional data from Kaggle was merged using common keys (`title` and `English`), standardizing values to avoid mismatches.
6. **Publication:** Results and code were published to a GitHub repository (`BigDataFinal`), showcasing reproducibility and transparency.
7. **Validation:** The data schema was validated to ensure consistency with expectations.


#Code

##Prep

from google.colab import auth
auth.authenticate_user()

print("Authenticated successfully.")

!pip install pymongo

!pip install google-cloud-storage pyspark pymongo google-cloud-bigquery

# Import Libraries
from google.cloud import bigquery, storage
from pymongo import MongoClient
import pandas as pd
import matplotlib.pyplot as plt
from io import StringIO

##Step 1: Loading Data from GCS

# Step 1: Load Data from GCS
bucket_name = "my_anime_list"
file_path = "MyAnimeList/final_animedataset.csv"

try:
    # Initialize Google Cloud Storage Client
    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(file_path)
    data = blob.download_as_text()

    # Load CSV into DataFrame
    df = pd.read_csv(StringIO(data))
    print("Data loaded successfully from GCS.")
    print(df.head())
except Exception as e:
    print(f"Error loading data from GCS: {e}")


## Step 2: Data Cleaning and Transformation

try:
    # Handling missing values
    df = df.dropna(subset=['genre', 'score'])
    df['score'] = pd.to_numeric(df['score'], errors='coerce')
    df = df.dropna()

    print("Data cleaning completed.")
except Exception as e:
    print(f"Error during data cleaning: {e}")


## Step 3: Analysis


try:
    # Group by genre and calculate mean score
    df_grouped = df.groupby('genre')['score'].mean().sort_values(ascending=False).reset_index()
    print("Top genres by average score:")
    print(df_grouped.head(10))

    # Visualization
    genres = df_grouped['genre'].head(10)
    avg_scores = df_grouped['score'].head(10)

    plt.figure(figsize=(10, 6))
    plt.bar(genres, avg_scores, color='blue')
    plt.xlabel("Genres")
    plt.ylabel("Average Score")
    plt.title("Top Genres by Average Score")
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()
except Exception as e:
    print(f"Error during analysis: {e}")


# Step 4: Save Results to GCS

try:
    # Save results to GCS
    output_csv = df_grouped.to_csv(index=False)
    output_blob_path = "MyAnimeList/top_genres.csv"
    output_blob = bucket.blob(output_blob_path)
    output_blob.upload_from_string(output_csv, content_type="text/csv")
    print(f"Results saved to GCS at {output_blob_path}")
except Exception as e:
    print(f"Error saving results: {e}")

## Step 5: Load and Merge additional data from an external source (Kaggle.com)

!pip install kaggle


from google.colab import files
files.upload("/content/kaggle.json")

# Download the dataset
!kaggle datasets download -d bhavyadhingra00020/top-anime-dataset-2024 --unzip -p kaggle_data/


import os

# List files in the kaggle_data directory
files = os.listdir("kaggle_data/")
print("Files in kaggle_data directory:", files)

import pandas as pd

# Load the Kaggle dataset
try:
    kaggle_file_path = "kaggle_data/Top_Anime_data.csv"  # Adjust file name if needed
    kaggle_df = pd.read_csv(kaggle_file_path)
    print("Kaggle data loaded successfully.")
    print(kaggle_df.head())
except Exception as e:
    print(f"Error loading Kaggle dataset: {e}")

print("Columns in df:", df.columns)
print("Columns in kaggle_df:", kaggle_df.columns)

# Merge datasets using different column names
try:
    # Standardize column values to lower case and strip whitespace for consistency
    df["title"] = df["title"].str.strip().str.lower()
    kaggle_df["English"] = kaggle_df["English"].str.strip().str.lower()

    # Merge using 'Title' from df and 'English' from kaggle_df
    merged_df = pd.merge(df, kaggle_df, left_on="title", right_on="English", how="inner")
    print("Datasets merged successfully.")
    print(merged_df.head())
except Exception as e:
    print(f"Error merging datasets: {e}")


try:
    # Save merged DataFrame to a CSV file in /content directory
    output_file_path = "/content/merged_data.csv"
    merged_df.to_csv(output_file_path, index=False)
    print(f"Merged DataFrame saved successfully to {output_file_path}")
except Exception as e:
    print(f"Error saving merged DataFrame: {e}")


##Step 6: Publish Code and results to GitHub.

!git clone https://github.com/EmilyBartman/BigDataFinal.git

!ls /content

import os
from git import Repo

try:
    # Define the data path and repository directory
    data_path = "/content/merged_data.csv"  # Replace with your notebook's actual name
    repo_dir = "/content/BigDataFinal"

    # Ensure the repository is cloned
    if not os.path.exists(repo_dir):
        remote_url = "https://github.com/EmilyBartman/BigDataFinal.git"
        Repo.clone_from(remote_url, repo_dir)
        print(f"Cloned repository to {repo_dir}")
    else:
        print(f"Repository already exists at {repo_dir}")

    # Check if the data exists
    if os.path.exists(data_path):
        os.system(f"cp {data_path} {repo_dir}/")
        print(f"Copied {data_path} to {repo_dir}/")
    else:
        raise FileNotFoundError(f"Notebook {data_path} does not exist.")

    # Initialize the repository
    repo = Repo(repo_dir)

    # Set the remote URL with your Personal Access Token (PAT)
    origin = repo.remote(name="origin")
    origin.set_url("https://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@github.com/EmilyBartman/BigDataFinal.git")

    # Stage, commit, and push changes
    repo.git.add(all=True)
    repo.index.commit("Update: Added current .ipynb file and results")
    origin.push(refspec="HEAD:main")  # Push to the main branch
    print("Merged Data is published to GitHub.")
except Exception as e:
    print(f"Error publishing to GitHub: {e}")


## Step 7: Validate Data Schema

expected_columns = ["genre", "score"]
if not all(col in df.columns for col in expected_columns):
    raise ValueError(f"DataFrame does not have the expected columns: {expected_columns}")

# Document Data Schema
schema = {col: str(df[col].dtype) for col in df.columns}
print("Data schema:", schema)

#Results
The analysis revealed several key insights:
1. **Data Cleaning:** Successfully removed rows with missing or invalid data, resulting in a clean dataset with well-defined `genre` and `score` columns.
2. **Genre Analysis:**
   - Genres with the highest average scores were identified, with the top genres being action, drama, and thriller.
   - Visualization using a bar chart highlighted these findings.
3. **Data Augmentation:** The Kaggle dataset provided complementary attributes, such as English titles, further enriching the analysis.
4. **Storage and Sharing:**
   - The results were saved back to GCS for long-term storage.
   - All findings, along with the code, were pushed to GitHub for public access.


#Discussion

### **Interpretation of Results**
The analysis revealed that genres like action and thriller tend to score highly on average, suggesting a preference among viewers for dynamic and emotionally engaging content. This insight can guide content creators in targeting audience preferences.

### **Application of Skills**
The project applied several key skills from the course, including:
- Cloud storage integration using GCS.
- Data wrangling and cleaning techniques in Python.
- Use of visualization libraries for storytelling with data.
- Employing APIs for dataset augmentation (Kaggle).
- Collaborative coding practices by publishing to GitHub.

### **Challenges and Barriers**
- **Integration Issues:** Merging datasets required careful standardization of column values, which introduced complexity.
- **Cloud Permissions:** Initial issues with GCS permissions required troubleshooting and re-authentication.
- **Dataset Size:** The Kaggle dataset had a different structure and required alignment with the primary dataset, demanding additional preprocessing.

### **Connection to Course Modules**
This project connects directly to the course modules on:
1. **Big Data Infrastructure:** Leveraging GCP and scalable tools.
2. **Data Analysis and Cleaning:** Employing Pandas for robust manipulation.
3. **Visualization:** Using Matplotlib for actionable insights.
4. **Data Augmentation:** Integrating external datasets to enrich findings.


# Conclusion
The project successfully demonstrated the end-to-end workflow of a data analysis pipeline in a cloud-based environment. Insights from the anime dataset highlighted popular genres and audience preferences. The integration of skills from the course, alongside the utilization of cloud and virtualized tools, underscores the effectiveness of modern data science practices.


#References
1.   Datasets:
  *   MyAnimeList Dataset: https://www.kaggle.com/datasets/dbdmobile/myanimelist-dataset 
  *   Top Anime Dataset 2024: https://www.kaggle.com/datasets/bhavyadhingra00020/top-anime-dataset-2024
2.   Google Cloud Documentation:
  *   Google Cloud Storage: https://cloud.google.com/storage/docs
  *   Google BigQuery: https://cloud.google.com/bigquery/docs
3.   MongoDB Documentation:
  *   MongoDB NoSQL Database: https://www.mongodb.com/docs/
4.   Apache Spark Documentation:
  *   Apache Spark: https://spark.apache.org/docs/latest/
5.   Python Libraries:
  *   Pandas: https://pandas.pydata.org/docs/
  *   Matplotlib: https://matplotlib.org/stable/users/index.html
  *   Seaborn: https://seaborn.pydata.org/
  *   Scikit-learn: https://scikit-learn.org/stable/user_guide.html
6.   Google Colab Authentication:
  *   Authenticating Colab with GCP: https://colab.research.google.com/notebooks/io.ipynb
7.   Course Notes:
  *   Big Data Concepts and Applications: https://iu.instructure.com/courses/2252964/modules
8.   GitHub Repo: 
  *   BigDataFinal: https://github.com/EmilyBartman/BigDataFinal
