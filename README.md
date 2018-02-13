# coding: utf-8

# # Data Set Up

# In[30]:


#import dependencies
import pandas as pd
import os


# In[31]:


#read raw data csv files
students_csv = "raw_data\students_complete.csv"
schools_csv = "raw_data\schools_complete.csv"


# In[32]:


#create data frames
students_df = pd.read_csv(students_csv)
schools_df=pd.read_csv(schools_csv)


# In[33]:


#merge two data frames
schools_df.rename(columns = {'name': 'school'}, inplace = True)
merged_df = students_df.merge(schools_df, how = 'left', on = 'school')


# # District Summary

# In[34]:


#create array of unique school names
unique_school_names = schools_df['school'].unique()

#gives the length of unique school names to give us how many schools
school_count = len(unique_school_names)

#district student count
dist_student_count = schools_df['size'].sum()

#student count from student file (to verify with district student count)
total_student_rec = students_df['name'].count()

#total budget
total_budget = schools_df['budget'].sum()

#calculations for number and % passing reading
num_passing_reading = students_df.loc[students_df['reading_score'] >= 70]['reading_score'].count()
perc_pass_reading = num_passing_reading/total_student_rec

#calculations for number and % passing math
num_passing_math = students_df.loc[students_df['math_score'] >= 70]['math_score'].count()
perc_pass_math = num_passing_math/total_student_rec

#average math score calculation
avg_math_score = students_df['math_score'].mean()

#average reading score calculation
avg_reading_score = students_df['reading_score'].mean()

#Overall Passing Rate Calculations
overall_pass = students_df[(students_df['math_score'] >= 70) & (students_df['reading_score'] >= 70)]['name'].count()/total_student_rec


# In[36]:


#create district dataframe
district_summary = pd.DataFrame({
    "Total Schools": [school_count],
    "Total Students": [dist_student_count],
    "Total Budget": [total_budget],
    "Average Reading Score": [avg_reading_score],
    "Average Math Score": [avg_math_score],
    "% Passing Reading":[perc_pass_reading],
    "% Passing Math": [perc_pass_math],
    "Overall Passing Rate": [overall_pass]})

#format cells
dist_sum.style.format({"Total Budget": "${:,.2f}", 
                       "Average Reading Score": "{:.1f}", 
                       "Average Math Score": "{:.1f}", 
                       "Total Students": "{:.0f}", 
                       "% Passing Math": "{:.1%}", 
                       "% Passing Reading": "{:.1%}", 
                       "Overall Passing Rate": "{:.1%}"})


# # School Summary

# In[38]:


#groups by school
by_school = merged_df.set_index('school').groupby(['school'])

#school types
sch_types = schools_df.set_index('school')['type']

# total students by school
stu_per_sch = by_school['Student ID'].count()

#school budget
sch_budget = schools_df.set_index('school')['budget']

#per student budget
stu_budget = schools_df.set_index('school')['budget']/schools_df.set_index('school')['size']

#avg scores by school
avg_math = by_school['math_score'].mean()
avg_read = by_school['reading_score'].mean()

# % passing scores
pass_math = merged_df[merged_df['math_score'] >= 70].groupby('school')['Student ID'].count()/stu_per_sch 
pass_read = merged_df[merged_df['reading_score'] >= 70].groupby('school')['Student ID'].count()/stu_per_sch 
overall = merged_df[(merged_df['reading_score'] >= 70) & (merged_df['math_score'] >= 70)].groupby('school')['Student ID'].count()/stu_per_sch 


# In[41]:


#Create school dataframe
sch_summary = pd.DataFrame({
    "School Type": sch_types,
    "Total Students": stu_per_sch,
    "Per Student Budget": stu_budget,
    "Total School Budget": sch_budget,
    "Average Math Score": avg_math,
    "Average Reading Score": avg_read,
    '% Passing Math': pass_math,
    '% Passing Reading': pass_read,
    "Overall Passing Rate": overall})

#formatting
sch_summary.style.format({'Total Students': '{:,}', 
                          "Total School Budget": "${:,}", 
                          "Per Student Budget": "${:.0f}",
                          'Average Math Score': "{:.1f}", 
                          'Average Reading Score': "{:.1f}", 
                          "% Passing Math": "{:.1%}", 
                          "% Passing Reading": "{:.1%}", 
                          "Overall Passing Rate": "{:.1%}"})


# # Top Performing Schools by Passing Rate

# In[16]:


#sort values by passing rate and then only print top 5 
top_5 = sch_summary.sort_values("Overall Passing Rate", ascending = False)
top_5.head().style.format({'Total Students': '{:,}',
                           "Total School Budget": "${:,}", 
                           "Per Student Budget": "${:.0f}", 
                           "% Passing Math": "{:.1%}", 
                           "% Passing Reading": "{:.1%}", 
                           "Overall Passing Rate": "{:.1%}"})


# # Bottom Perfoming Schools by Passing Rate

# In[18]:


#bottom 5 schools from worse to best
bottom_5 = top_5.tail()
bottom_5 = bottom_5.sort_values('Overall Passing Rate')
bottom_5.style.format({'Total Students': '{:,}', 
                       "Total School Budget": "${:,}", 
                       "Per Student Budget": "${:.0f}", 
                       "% Passing Math": "{:.1%}", 
                       "% Passing Reading": "{:.1%}", 
                       "Overall Passing Rate": "{:.1%}"})


# # Math Scores by Grade

# In[42]:


#creates grade level average math scores for each school 
ninth_math = students_df.loc[students_df['grade'] == '9th'].groupby('school')["math_score"].mean()
tenth_math = students_df.loc[students_df['grade'] == '10th'].groupby('school')["math_score"].mean()
eleventh_math = students_df.loc[students_df['grade'] == '11th'].groupby('school')["math_score"].mean()
twelfth_math = students_df.loc[students_df['grade'] == '12th'].groupby('school')["math_score"].mean()


# In[43]:


#create dataframe
math_scores = pd.DataFrame({
        "9th": ninth_math,
        "10th": tenth_math,
        "11th": eleventh_math,
        "12th": twelfth_math})
math_scores = math_scores[['9th', '10th', '11th', '12th']]
math_scores.index.name = "School"

#format
math_scores.style.format({'9th': '{:.1f}', 
                          "10th": '{:.1f}', 
                          "11th": "{:.1f}", 
                          "12th": "{:.1f}"})


# # Reading Scores by Grade

# In[45]:


#creates grade level average reading scores for each school
ninth_reading = students_df.loc[students_df['grade'] == '9th'].groupby('school')["reading_score"].mean()
tenth_reading = students_df.loc[students_df['grade'] == '10th'].groupby('school')["reading_score"].mean()
eleventh_reading = students_df.loc[students_df['grade'] == '11th'].groupby('school')["reading_score"].mean()
twelfth_reading = students_df.loc[students_df['grade'] == '12th'].groupby('school')["reading_score"].mean()


# In[44]:


#create dataframe
reading_scores = pd.DataFrame({
        "9th": ninth_reading,
        "10th": tenth_reading,
        "11th": eleventh_reading,
        "12th": twelfth_reading})
reading_scores = reading_scores[['9th', '10th', '11th', '12th']]
reading_scores.index.name = "School"

#format
reading_scores.style.format({'9th': '{:.1f}', 
                             "10th": '{:.1f}', 
                             "11th": "{:.1f}", 
                             "12th": "{:.1f}"})


# # Scores by School Spending

# In[46]:


#create spending bins
bins = [0, 584.999, 614.999, 644.999, 999999]
group_name = ['< $585', "$585 - 614", "$615 - 644", "> $644"]
merged_df['spending_bins'] = pd.cut(merged_df['budget']/merged_df['size'], bins, labels = group_name)

#group by spending
by_spending = merged_df.groupby('spending_bins')

#calculations
avg_math = by_spending['math_score'].mean()
avg_read = by_spending['reading_score'].mean()
pass_math = merged_df[merged_df['math_score'] >= 70].groupby('spending_bins')['Student ID'].count()/by_spending['Student ID'].count()
pass_read = merged_df[merged_df['reading_score'] >= 70].groupby('spending_bins')['Student ID'].count()/by_spending['Student ID'].count()
overall = merged_df[(merged_df['reading_score'] >= 70) & (merged_df['math_score'] >= 70)].groupby('spending_bins')['Student ID'].count()/by_spending['Student ID'].count()


# In[49]:


#create dataframe         
scores_by_spend = pd.DataFrame({
    "Average Math Score": avg_math,
    "Average Reading Score": avg_read,
    '% Passing Math': pass_math,
    '% Passing Reading': pass_read,
    "Overall Passing Rate": overall})

#reorder columns to match output example
scores_by_spend = scores_by_spend[[
    "Average Math Score",
    "Average Reading Score",
    '% Passing Math',
    '% Passing Reading',
    "Overall Passing Rate"]]

scores_by_spend.index.name = "Per Student Budget"
scores_by_spend = scores_by_spend.reindex(group_name)

#formating
scores_by_spend.style.format({'Average Math Score': '{:.1f}', 
                              'Average Reading Score': '{:.1f}', 
                              '% Passing Math': '{:.1%}', 
                              '% Passing Reading':'{:.1%}', 
                              'Overall Passing Rate': '{:.1%}'})


# # Scores by School Size

# In[50]:


#create size bins
bins = [0, 999, 1999, 99999999999]
group_name = ["Small (<1000)", "Medium (1000-2000)" , "Large (>2000)"]
merged_df['size_bins'] = pd.cut(merged_df['size'], bins, labels = group_name)

#group by spending
by_size = merged_df.groupby('size_bins')

#calculations 
avg_math = by_size['math_score'].mean()
avg_read = by_size['math_score'].mean()
pass_math = merged_df[merged_df['math_score'] >= 70].groupby('size_bins')['Student ID'].count()/by_size['Student ID'].count()
pass_read = merged_df[merged_df['reading_score'] >= 70].groupby('size_bins')['Student ID'].count()/by_size['Student ID'].count()
overall = merged_df[(merged_df['reading_score'] >= 70) & (merged_df['math_score'] >= 70)].groupby('size_bins')['Student ID'].count()/by_size['Student ID'].count()


# In[51]:


#create dataframe           
scores_by_size = pd.DataFrame({
    "Average Math Score": avg_math,
    "Average Reading Score": avg_read,
    '% Passing Math': pass_math,
    '% Passing Reading': pass_read,
    "Overall Passing Rate": overall})
            
#reorder columns to match output example
scores_by_size = scores_by_size[[
    "Average Math Score",
    "Average Reading Score",
    '% Passing Math',
    '% Passing Reading',
    "Overall Passing Rate"]]

scores_by_size.index.name = "Total Students"
scores_by_size = scores_by_size.reindex(group_name)

#formatting
scores_by_size.style.format({'Average Math Score': '{:.1f}', 
                              'Average Reading Score': '{:.1f}', 
                              '% Passing Math': '{:.1%}', 
                              '% Passing Reading':'{:.1%}', 
                              'Overall Passing Rate': '{:.1%}'})


# # Scores by School Type

# In[56]:


# group by type of school
by_type = merged_df.groupby("type")

#find average score by type
avg_scores_by_type = by_type['math_score', 'reading_score'].mean().reset_index()

#find # passing by type of school
pass_read_by_type = merged_df[merged_df['reading_score'] >= 70].groupby("type")['reading_score'].count().reset_index()
pass_read_by_type.rename(columns = {'reading_score': '# pass reading'}, inplace=True)

pass_math_by_type = merged_df[merged_df['math_score'] >= 70].groupby("type")['math_score'].count().reset_index()
pass_math_by_type.rename(columns = {'math_score': '# pass math'}, inplace=True)

#find number of students by type of school
size_by_type = by_type['name'].count().reset_index()
size_by_type.rename(columns = {'name':'size'}, inplace = True)


# In[55]:


#merge data for calculations
scores_by_type = pd.merge(avg_scores_by_type, pass_read_by_type, on = "type").merge(pass_math_by_type, on="type").merge(size_by_type, on="type")
scores_by_type['% Passing Reading'] = scores_by_type['# pass reading']/scores_by_type['size']
scores_by_type['% Passing Math'] = scores_by_type['# pass math']/scores_by_type['size']

# only keep needed columns
scores_by_type = scores_by_type[["type", 'math_score', 'reading_score', '% Passing Math', '% Passing Reading']]

# calc passing rate for each type
scores_by_type['Overall Passing Rate'] = (scores_by_type['% Passing Reading']+ scores_by_type['% Passing Math'])/2

#formatting
scores_by_type.rename(columns = {"type": "School Size",'math_score': 'Average Math Score', 'reading_score':'Average Reading Score'}, inplace=True)
scores_by_type.set_index('School Size', inplace=True)
scores_by_type.style.format({'Average Math Score': '{:.1f}', 'Average Reading Score': '{:.1f}', '% Passing Math': '{:.1%}', '% Passing Reading':'{:.1%}', 'Overall Passing Rate': '{:.1%}'})


# # Academy of Py Analysis

# -Charter schools appear to out perform District schools.  Top 5 schools are Charter, while the bottom 5 are District. 

# -Schools with less than 2000 students appear to have higher passing rates. Small and Medium sized schools perform better in passing rates than Large Schools.

# -Bottom performing schools spend more money per student than the top performing.

# -Spending more than $615 per student does not appear to have much effect on the passing rates, even showing a decrease in grades as spending per student increases.
