# INT375-CA2

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns


df = pd.read_excel(r"C:\Users\ASUS\Downloads\Hospital_General_Information.xlsx")

print("Data Info:")
print(df.info())

print("Missing Values per Column:")
print(df.isnull().sum())

df_dropped = df.dropna()
df_filled = df.fillna({
    'Hospital Name': 'Unknown',
    'City': 'Unknown',
    'ZIP Code': 0,
    'Phone Number': 'Not Available'
})
df_filled.to_excel(r"C:\Users\ASUS\Downloads\Hospital_Cleaned.xlsx", index=False)
print("Cleaned data saved as 'Hospital_Cleaned.xlsx'\n")

#----------------------------

#heatmap
numeric_cols = df.select_dtypes(include='number').columns
plt.figure(figsize=(10, 8))
corr = df[numeric_cols].corr()

sns.heatmap(corr, annot=True, fmt=".2f", cmap='coolwarm', square=True, cbar_kws={"shrink": .75})
plt.title("Correlation Heatmap of Numeric Features")
plt.tight_layout()
plt.show()

sns.set(style="whitegrid")

# histogram
# Distribution of ZIP Codes
plt.figure(figsize=(8, 5))
df['ZIP Code'].dropna().astype(int).plot(kind='hist', bins=20, color='skyblue', edgecolor='black')
plt.title('Histogram of ZIP Codes')
plt.xlabel('ZIP Code')
plt.ylabel('Frequency')
plt.grid(True)
plt.show()

# bargraph
# Count of hospitals by State (top 10)

top_states = df['State'].value_counts().nlargest(10)

plt.figure(figsize=(10, 6))
sns.barplot(
    x=top_states.index,
    y=top_states.values,
    hue=top_states.index,
    palette='viridis',
    legend=False
)
plt.title('Top 10 States by Number of Hospitals')
plt.xlabel('State')
plt.ylabel('Number of Hospitals')
plt.show()

# Scatter plot of Hospital overall rating vs emergency services
df['Hospital overall rating'] = pd.to_numeric(df['Hospital overall rating'], errors='coerce')
df_clean = df.dropna(subset=['Hospital overall rating', 'Emergency Services'])

plt.figure(figsize=(8, 6))
sns.scatterplot(
    data=df_clean,
    x='Hospital overall rating',
    y='Emergency Services',
    hue='State',
    palette='tab10',
    alpha=0.7
)
plt.title('Scatter Plot of Rating vs Emergency Services')
plt.xlabel('Hospital Overall Rating')
plt.ylabel('Emergency Services (1 = Yes, 0 = No)')
plt.legend(loc='upper right', bbox_to_anchor=(1.3, 1))
plt.show()

# box plot
# Boxplot of hospital ratings by Ownership type
plt.figure(figsize=(12, 6))
sns.boxplot(
    data=df,
    x='Hospital Ownership',
    y='Hospital overall rating',
    hue='Hospital Ownership',
    palette='Set2',
    legend=False
)
plt.xticks(rotation=45)
plt.title('Box Plot of Hospital Ratings by Ownership Type')
plt.xlabel('Hospital Ownership')
plt.ylabel('Hospital Overall Rating')
plt.tight_layout()
plt.show()

#line plot
# Group by state and calculate average rating
# Calculate average rating for top 10 states
top_states_list = df['State'].value_counts().nlargest(10).index
df_top_states = df[df['State'].isin(top_states_list)]

avg_rating_by_state = df_top_states.groupby('State')['Hospital overall rating'].mean().sort_values()


plt.figure(figsize=(10, 6))
sns.lineplot(x=avg_rating_by_state.index, y=avg_rating_by_state.values, marker='o', color='purple')
plt.title('Average Hospital Rating by Top 10 States')
plt.xlabel('State')
plt.ylabel('Average Hospital Overall Rating')
plt.grid(True)
plt.tight_layout()
plt.show()



#-------------------------------------


#T-test

from scipy.stats import ttest_ind

df['Hospital overall rating'] = pd.to_numeric(df['Hospital overall rating'], errors='coerce')
df_ttest = df.dropna(subset=['Hospital overall rating', 'Emergency Services'])

group_emergency = df_ttest[df_ttest['Emergency Services'] == 'Yes']['Hospital overall rating']
group_no_emergency = df_ttest[df_ttest['Emergency Services'] == 'No']['Hospital overall rating']

t_stat, p_value = ttest_ind(group_emergency, group_no_emergency, equal_var=False)

print("T-test comparing hospital ratings between Emergency vs Non-Emergency hospitals:")
print(f"T-statistic: {t_stat:.4f}")
print(f"P-value: {p_value:.4f}")

if p_value < 0.05:
    print("Significant difference in ratings (p < 0.05)\n")
else:
    print("No significant difference in ratings (p ≥ 0.05)\n")


#chi-square test
    
from scipy.stats import chi2_contingency

contingency_table = pd.crosstab(df['Hospital Ownership'], df['Emergency Services'])
print("Contingency Table:")
print(contingency_table)

chi2_stat, p_val, dof, expected = chi2_contingency(contingency_table)

print("Chi-Square Test between 'Hospital Ownership' and 'Emergency Services':")
print(f"Chi-square Statistic: {chi2_stat:.4f}")
print(f"P-value: {p_val:.4f}")
print(f"Degrees of Freedom: {dof}")
print("Expected Frequencies Table:\n", expected)

if p_val < 0.05:
    print("Significant association between variables (p < 0.05)")
else:
    print("No significant association between variables (p ≥ 0.05)")
