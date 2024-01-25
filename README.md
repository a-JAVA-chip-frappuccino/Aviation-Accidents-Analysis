### Overview

In order to better know how to minimize the number of incidents, particularly fatal ones, when flying aircraft, we will be looking into past analytics from the National Transportation Safety Board. This comprises civil aviation accident records from 1962-2023. Then we will discuss some recommendations for how to extrapolate past data into preventing present and future incidents.

Although we cannot necessarily answer every question regarding how incidents occur, we can consider some elements of the dataset to consider what might be leading to incidents to then rectify them.

### Dataset

```
df = pd.read_csv('data/Aviation_Data.csv', dtype = {'Event.Id' : str}, low_memory = False)
```

The dataset columns:
- event ID
- investigation type
- accident number
- event date
- location
- country
- latitude
- longitude
- airport code
- airport name
- injury severity
- aircraft damage
- aircraft category
- registration number
- make
- model
- amateur-built
- number of engines
- engine type
- FAR description
- schedule
- purpose of flight
- air carrier
- total fatal injuries
- total serious injuries
- total minor injuries
- total uninjured
- weather condition
- broad phase of flight
- report status
- publication date



### Data Analysis

Let's start by cleaning up the list of fatal crashes to group injury severity column into fatal, non-fatal, and misc. injuries.

```
fatal_rows = ['Fatal']

unique_values = set(list(df['Injury.Severity']))

for value in unique_values:
    if type(value) == str:
        if value != 'Non-Fatal' and 'Fatal' in value:
            fatal_rows.append(value)

df['Injury.Severity'] = df['Injury.Severity'].apply(lambda value: "Fatal" if value in fatal_rows else value)
```

Then, let's separate the amateur and professional craft into two dataframes, and sum the number of rows under each descriptor.

```
df_amateur = df[df['Amateur.Built'] == 'No']

df_prof = df[df['Amateur.Built'] == 'Yes']
```

```
total_amateur = df_amateur['Event.Id'].count()

total_prof = df_prof['Event.Id'].count()
```

Then plot the total fatal accidents by amateur aircraft versus professional aircraft.

```
fig, ax = plt.subplots()

ax.bar("Amateur Aircraft", total_amateur_fatal, color = 'darkblue')

ax.bar("Professional Aircraft", total_prof_fatal, color = 'darkblue')

ax.set_title("Fatal Incidents by Aircraft Build Type")
ax.set_ylabel("Total Fatal Incidents")

plt.show()
```



There appears to be more fatal incidents involving amateur aircraft than professional aircraft, likely due to the lack of vigilant oversight that goes into professional flights, such as rigorous pilot training and extensive flight and ground crews.

In order to have safer, less fatal flights, a strong recommendation would be to utilize professional crafts and all the additional benefits that come with these, such as ground crews and dual-pilot crews.



### Another point of consideration could be the number of engines of a plane. In general, single-engine planes are considered tiny (or "prop" planes), and larger planes range from regional flights from major airports to enormous commercial jets flying internationally.

```
df['Number.of.Engines'].value_counts()
```

### Looking at the engine counts, we will combine 1- and 0-engine craft into "small" planes (as these are better for shorter distances, with the latter even reflecting glider craft that do not gain height substantially). The remaining planes can be considered "large".

```
# combine all one- and two-engine planes into "small planes" category

df['Number.of.Engines'] = df['Number.of.Engines'].apply(lambda value: "Small" if (value == 0.0 or value == 1.0) else value)

df['Number.of.Engines'] = df['Number.of.Engines'].apply(lambda value: "Large" if (value == 2.0 or value == 3.0 or value == 4.0 or value == 6.0 or value == 8.0) else value)

# group planes after cleaning data

df_small = df[df['Number.of.Engines'] == 'Small']

df_large = df[df['Number.of.Engines'] == 'Large']
```

```
df['Number.of.Engines'].value_counts()
```

### Now let's tally up the small craft fatal incidents in comparison to the large craft fatal incidents.

```
total_small_fatal = df_small[df_small['Injury.Severity'] == 'Fatal']['Event.Id'].count()

total_large_fatal = df_large[df_large['Injury.Severity'] == 'Fatal']['Event.Id'].count()
```

```
fig, ax = plt.subplots()

ax.bar("Few-Engine Aircraft", total_small_fatal, color = 'goldenrod')

ax.bar("Multiple-Engine Aircraft", total_large_fatal, color = 'goldenrod')

ax.set_title("Fatal Incidents by Aircraft Engine Number")
ax.set_ylabel("Total Fatal Incidents")

plt.show()
```

A somewhat similar conclusion can be gleaned from this data as from the previous set. The fewer engines an aircraft has, the more likely that plane is to be involved in a crash. This could be explained by multi-engine craft having more trained flight crew and pilots, as well as to the inherent "redundancies" built into multiple engines (if one engine is lost, additional engines can temporarily keep the plane afloat if need be).

A recommendation from this data would be to once again utilize larger aircraft for flight in order to reduce the number of fatal incidents, as well as to prioritize multi-engine craft over glider and prop planes.



### As a final conclusion, let us examine the number of incidents based on the weather conditions behind flights.

```
df['Weather.Condition'].value_counts()
```

### To summarize these, VMC means Visual Meterological Conditions, or conditions in which it is deemed safe to fly by sight alone. IMC refers to Instrument Meterological Conditions, in which instruments should be used to assist in flight (visibility is reduced). UNK and Unk both refer to unknown conditions.

### To start cleaning the data, we should combine the final two conditions, as they refer to the same thing (unknown weather condition data).

```
# apply change to column

df['Weather.Condition'] = df['Weather.Condition'].apply(lambda value : 'UNK' if value == 'UNK' or value == 'Unk' else value)
```

```
# tally number of each weather condition (including unknown)

total_vmc = df['Weather.Condition'].value_counts()['VMC']

total_imc = df['Weather.Condition'].value_counts()['IMC']

total_unk = df['Weather.Condition'].value_counts()['UNK']
```

```
fig, ax = plt.subplots()

fig.set_figwidth(10)

ax.bar("Visual Meterological Conditions", total_vmc, color = 'darkred')

ax.bar("Instrument Meterological Conditions", total_imc, color = 'darkred')

ax.set_title("Incidents by Weather Condition Type")
ax.set_ylabel("Total Number of Crashes")

plt.show()
```

### From this information, it becomes immediately apparent that VMC conditions allow for far, far more crashes than IMC conditions (and unknown conditions make up a miniscule percentage of the data). Although this might not make immediate sense, as VMC conditions would be preferable, it could be ascertained that such conditions cause a "false sense of security" or perhaps even a lack of reliance on tools to assist in flying.

VMC conditions appear to contribute to far more crashes than IMC conditions. One probable explanation for this is that the instrumentation required to fly under IMC conditions allows for safer flying, and that simply relying upon visual guidelines for flying is more dangerous.

Therefore, utilizing instruments and tools while flying is a great way to reduce accidents.

### Final Conclusions

In conclusion, to reduce the number of incidents, and the probability that those incidents are fatal, it is recommended to utilize large, professional aircraft and to take full advantage of the robust instrumentation that comes with such craft.