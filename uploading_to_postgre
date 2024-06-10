
import pandas as pd
from sqlalchemy import create_engine
import os
conn_string = r'postgresql://postgres:password@localhost/unicef_sql_project'
db = create_engine(conn_string)
conn = db.connect()

files = ['demographics.csv', 'childhood_development.csv', 'social_protection.csv','child_mortality.csv']
path=r'C:\Users\DELL\Downloads'
for file in files:
    file_path=os.path.join(path, file)
    df = pd.read_csv(file_path)
    df.to_sql(file, con=conn, if_exists='replace', index=False)


