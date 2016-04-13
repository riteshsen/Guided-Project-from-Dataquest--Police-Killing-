# Guided-Project-from-Dataquest--Police-Killing-
#This repository stores the code to analyze the data about Police Killings in US and showcase any relationship of the killings with other #demographic data

import pandas as pd
import matplotlib.pyplot as plt

%matplotlib inline
police_killings=pd.read_csv("police_killings.csv",encoding="ISO-8859-1")

# Plotting Killings by ethnicity

plt.bar(range(6), police_killings["raceethnicity"].value_counts())
plt.xticks(range(6),["White","Black","Hispanic/Latino","Unknown","Asian/Pacific Islander","Native American"],rotation="vertical",ha="right")
plt.show()

#It looks like people identified as Black are far overrepresented in the shootings versus in the population of the US (28% vs 16%). 
#People identified as Hispanic appear to be killed about as often as random chance would account for (14% of the people killed as Hispanic, #versus 17% of the overall population).
#Whites are underrepresented among shooting victims vs their population percentage, as are Asians.


# Plotting Killings by income

income = police_killings["p_income"][police_killings["p_income"] != "-"]

income=income.astype(float)

income.hist(bins=40)

## Reading the state Population Data

state_pop = pd.read_csv("state_population.csv")

counts = police_killings["state_fp"].value_counts()


# Creating a new DataFrame to store the statewise count of killings

states = pd.DataFrame({"STATE": counts.index, "shootings": counts})

# Merging the two DFs 'states' & 'state_pop' to include the killings in each state
states = states.merge(state_pop,on="STATE")


states["pop_millions"] = states["POPESTIMATE2015"]/1000000

states["rate"] = states["shootings"]/states["pop_millions"]
states = states.sort_values("NAME",axis=0)

# Plotting shooting rate against each state
plt.bar(states["STATE"],states["rate"])
plt.show()

# removing rows with missing data 
pk=police_killings
pk=pk[(pk["share_white"] != '-') & 
      (pk["share_black"] != '-') &
      (pk["share_hispanic"]!='-')]

pk["share_white"]=pk["share_white"].astype(float)
pk["share_black"]=pk["share_black"].astype(float)
pk["share_hispanic"]=pk["share_hispanic"].astype(float)

# finding 10 states each with highest & lowest shooting rates

states = states.sort_values("rate",axis=0)
states["STATE"]=states["STATE"].astype(int)


top_10_crime_rate_states=[]
least_10_crime_rate_states=[]

for state_id in (states.head(10)["STATE"]): 
    least_10_crime_rate_states.append(state_id)
for state_id in (states.tail(10)["STATE"]): 
    top_10_crime_rate_states.append(state_id)
    
print (least_10_crime_rate_states)
print (top_10_crime_rate_states)

#finding any relationship with the shooting rates in each state with other columns in the 'police_killings' data
low_rate=pk[pk["state_fp"].isin(least_10_crime_rate_states)]
high_rate=pk[pk["state_fp"].isin(top_10_crime_rate_states)]

print (low_rate["state_fp"].head(5))
print (high_rate["state_fp"].head(5))
print (low_rate.mean())
print (high_rate.mean())

#It looks like the states with low rates of shootings tend to have a higher proportion of blacks in the population, and a lower proportion of #hispanics in the census regions where the shootings occur. It looks like the income of the counties where the shootings occur is higher.
#States with high rates of shootings tend to have high hispanic population shares in the counties where shootings occur.
