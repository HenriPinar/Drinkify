# Import all relevant libraries 
import streamlit as st
import numpy as np 
import pandas as pd 
import matplotlib.pyplot as plt 
import re
import math
import streamlit.components.v1 as components
import webbrowser
from collections import Counter
from googlesearch import search
import webbrowser
import time
from sklearn.linear_model import LinearRegression
import requests

# Streamlit Setup:

# Set Tab Title
st.set_page_config(
    page_title="Cocktail up your Life",
    page_icon="🍸",
)

# Import and set Logo for the Application
logo_image = "Logo.png"
st.image(logo_image, use_column_width=True)


# Set Title & Intro
st.write("<p style='text-align: center;'>Welcome to Cocktail up your life, the interactive application that turns your leftovers into delicious drinks or just gives you nearby bar locations, if you are to lazy to shake yourself!</p>", unsafe_allow_html=True)
st.markdown("<br>" * 1, unsafe_allow_html=True)

# Define Help button for the ingredient selection
help_button = '''
Select all the ingredients you have. Multiselect is possible.
'''.strip()


# Pre-Processing:

# Import static Dataframe for Drinks
df_drink = pd.read_csv("iba-cocktails-ingredients-web.csv", delimiter=",")

# Drop redundant Columns 
columns_to_drop = ["ingredient_direction", "note"]
df_drink = df_drink.drop(columns=columns_to_drop)

# Form a Dictionary with names (keys) and ingredients (values)
drink_list = df_drink.iloc[:, 1].tolist()
drink_dict = dict(Counter(drink_list))

# Define function to retrieve Weblinks for the missing ingredients
def search_products(products):
    first_results = {}

    for product in products:
        query = f"{product} kaufen"
        search_result = next(search(query, num=1, stop=1, pause=2), None)
        first_results[product] = search_result
    
    return first_results

# Setup sidebar to change views
view_options = ["Find Your Drink", "Add Your Creation", "Find a nearby Bar"]
selected_view = st.sidebar.radio("Select:", view_options)

# Paragraph
st.markdown("<br>" * 1, unsafe_allow_html=True)

# View 1 - add your creation:


if selected_view == "Add Your Creation":
    
    # Setup Title
    st.title("Add your own creation 🪄")
    
    # Initiate Text Input field for drink name
    name_add = st.text_input("Add your Drink Name:")

    # Initiate slider for ingredients count
    selected_count_ingredients = st.slider('How many ingredients does the drink have?', min_value=1, max_value=10, value=1)
    
    # Define empty list for user inputs
    ingredient_list = []
    quantity_list = []
    unit_list = []
    
    # Since some filds are not defined until the user input we use try and except to hide the errors that occur
    try: 
    
        # Set up for-lop to iterate over the inputs and put them into the corresponding lists
        def add_ingredient(ingredient_count):

            for i in range(ingredient_count):
                ingredient = st.text_input(f"Add your ingredient {i + 1}:", help = "Write with Capital letters e.g. Vodka")
                quantity = st.text_input(f"Add quantity for {ingredient}:", help = "Write numeric Values e.g. 12")
                unit = st.text_input(f"Add unit for {ingredient}:", help = "Write the abbreviations e.g. ml")
                st.markdown("<br>" * 2, unsafe_allow_html=True)
            
                ingredient_list.append(ingredient)
                quantity_list.append(quantity)
                unit_list.append(unit)

        # Use the function to set up all fields for the ingredient count        
        add_ingredient(selected_count_ingredients)

        # Form a dictionry for the inputs with corresponding column names as keys
        ingredient_data = []
    
        for ingredient, quantity, unit in zip(ingredient_list, quantity_list, unit_list):
            ingredient_data.append({
                'name': name_add,
                'ingredient': ingredient,
                'quantity': quantity,
                'unit': unit
            })
        
        # Append the existing Dataframe
        df_drink = df_drink.append(ingredient_data, ignore_index=True)
        
        # Due to the iteration the own crearion is added several times. We delete all those with nan values
        df_drink = df_drink.fillna('Own Creation')
        
        # Update the new dataframe within the original csv
        df_drink.to_csv("df_drink_4.csv", mode='a', index=False, header=False)


    except Exception as e:
        pass

    
# View 2 - Find your Drink:


elif selected_view == "Find Your Drink":
    
    # Setup title
    st.title("Find a Drink for your Leftovers 🔎")
    
    # Import the updated Dataframe
    df_drink = pd.read_csv("df_drink_4.csv", delimiter=",").astype(str)
    
    # Rename columns 
    df_drink.columns = ['category', 'name', 'quantity', "unit", "ingredient"]
    
    # Drop all dublicates coming from csv update in view 1
    df_drink = df_drink.drop_duplicates()

    # retrieve all ingredients one time and sort them alphabetically
    ingredients_options = sorted(df_drink.iloc[:, 4].unique())

    # Setup the multiselect and fill tem with the unique ingredients
    selected_options_ingredients = st.multiselect('What has your fridge to offer?', ingredients_options, help = help_button)

    # Paragraph
    st.markdown("<br>" * 1, unsafe_allow_html=True)

    # Setup slider to chose the people count 
    selected_options_people = st.slider('How many are you?', min_value=1, max_value=10, value=[1])
    
    # Save the selected ingredients in a list
    selected_options_ingredients = [ingredient for ingredient in selected_options_ingredients]
 
    # Setup columns to center certain content
    col1, col2, col3 = st.columns([1.5, 0.8, 1.5])
    with col2:
        if st.button('Find your drink 🔎', key='find_drink_button'):
        
            # Loading Spinner
            with st.spinner("Searching for drinks🍸"):
                time.sleep(1)
               
                # Filter the Dataframe with ingredeint input from the user
                filtered_drink = df_drink[df_drink['ingredient'].isin(selected_options_ingredients)]
            
                # Covert and drop nan values
                filtered_drink.replace({"nan": np.nan}, inplace=True)
                filtered_drink.dropna(inplace=True)
                df_drink.replace({"nan": np.nan}, inplace=True)
                df_drink.dropna(inplace=True)

                # Count frequencies for the names since every ingredient has a line and form a dictionary with count values as values
                filtered_drink_list = filtered_drink.iloc[:, 1].tolist()
                filtered_drink_dict = dict(Counter(filtered_drink_list))
                
                # Do the same with the Dataframe 
                drink_list = df_drink.iloc[:, 1].tolist()
                drink_dict = dict(Counter(drink_list))
    
                # Find common names between the whole drink Dataframe and the user input and put them into a new dictionary
                common_keys = drink_dict.keys() & filtered_drink_dict.keys()
                combined_dict = {key: drink_dict[key] for key in common_keys}
                 
                # Divide the full ingredient count per drink from the user input by the ingredeint count from the full Dataframe to find matching percentages
                result_dict = {}
                for key in filtered_drink_dict.keys():
                    result_dict[key] = filtered_drink_dict[key] / combined_dict[key]
            
                # Find the drink name with the biggest percentage
                matching_drink = max(combined_dict, key=lambda k: result_dict[k])
    
    # Setup output if no ingredients are chosen and scale it with the people count
    col1, col2, col3 = st.columns([1, 4.5, 1])
    with col2:
        if not selected_options_ingredients: 
                if selected_options_people[0] == 1:
                    st.write("We can only offer you a recipe for a glass of tap water at this point 🧊")
                else:
                    st.write(f"We can only offer you a recipe for {selected_options_people[0]} glasses of tap water at this point 🧊")

    try: 
  
        # Filter the Datafrme with the matching drink name
        df_filtered_matching_drink = df_drink[df_drink["name"] == matching_drink] 

        # Extract quantities for the matching drink and scale them with people count
        quantities_matching_drink = df_filtered_matching_drink.iloc[:, 2]
        quantities_matching_drink_scaled = quantities_matching_drink.astype(int) * selected_options_people
    
        # Update the Dataframe with the scaled quantities and loc quantities, units and ingredients
        df_filtered_matching_drink["quantities_scaled"] = quantities_matching_drink_scaled
        df_filtered_matching_drink = df_filtered_matching_drink.iloc[:, 3:]

        # Combine the scaled quantities with the corresponding unit
        quantities_with_units = df_filtered_matching_drink['quantities_scaled'].astype(str) + " " + df_filtered_matching_drink['unit']

        # Filter the ingredients for the matching drink
        ingrediets_matching_drink = df_filtered_matching_drink["ingredient"]
    
        # Centered bold output to display the matching drink name 
        centered_text_html = f"""
            <div style="display: flex; justify-content: center; align-items: center; height: 100vh;">
                <h1>{matching_drink}</h1>
            </div>
        """
        components.html(centered_text_html)
        
        # Output to display ingredients with quantities and units and add cocktail gifs 
        col1, col2, col3 = st.columns([1, 2, 1])
        with col2:
            for ingredient, quantity in zip(ingrediets_matching_drink, quantities_with_units):
                st.markdown(f"<div style='text-align: center;'><strong>{ingredient.title()}:</strong> {quantity}</div>", unsafe_allow_html=True)
        with col1:
            st.image("cocktail.gif", use_column_width=True)
        with col3:
            st.image("cocktail.gif", use_column_width=True)
        
        # Iterate over the ingredeints of the matching drinks and compare them to the input to find missing ingredients
        not_listed_ingredients = [ingredient for ingredient in ingrediets_matching_drink if
                                str(ingredient) not in selected_options_ingredients and not pd.isna(ingredient)]

        # Retrieve purchasing links for the missing ingredients
        link = search_products(not_listed_ingredients)
    
        # Setup individual output depending on the quantitiy of missing ingredients 
        if len(not_listed_ingredients) > 1:
            st.markdown("<div style='text-align: center;'><strong>The following ingredients are not in your fridge:</strong></div>", unsafe_allow_html=True)
        elif len(not_listed_ingredients) == 1:
            st.markdown("<div style='text-align: center;'><strong>The following ingredient is not in your fridge:</strong></div>", unsafe_allow_html=True)
        else:
            st.markdown("<div style='text-align: center;'><strong>You have all the ingredients. Have fun🎉</strong></div>", unsafe_allow_html=True)   
    
        # retrieve the first link and print them with the corresponding ingredient into an expander
        col1, col2, col3 = st.columns([1, 2, 1])
        for key, value in link.items():
            with col2:
                with st.expander(key):
                    st.write(f"<div style='text-align: center;'><a href='{value}' target='_blank'>{value}</a></div>", unsafe_allow_html=True)   
        
        # Form a new Dataframe with ingredients as columns and form boolean values for the occurance
        selected_columns = ["name", "ingredient"]
        df_drink_new = df_drink.loc[:, selected_columns]
        df_machine_learning = pd.pivot_table(df_drink_new, index='name', columns='ingredient', aggfunc=lambda x: 1, fill_value=0)
        df_machine_learning = df_machine_learning > 0

        # Pre-Process the new Dataframe
        df_machine_learning.reset_index(inplace=True)
        df_machine_learning.columns.name = None
        df_machine_learning.iloc[:, 1:] = df_machine_learning.iloc[:, 1:].astype(int)

        # Form a correlation matrix and create a Dataframe
        correlation_matrix = df_machine_learning.corr()
        coefficients_df_1 = pd.DataFrame(columns=correlation_matrix.columns)

        # Setup linear regression to forecast the correlation coefficients for each ingredient
        for ingredient in selected_options_ingredients:

            X = correlation_matrix.drop(columns=[ingredient])
            y = correlation_matrix[ingredient]

            model = LinearRegression()
            model.fit(X, y)
    
            temp_df = pd.DataFrame(model.coef_, index=X.columns, columns=[ingredient]).T

            coefficients_df_1 = pd.concat([coefficients_df_1, temp_df])

        # Extract the max correlation per row
        max_columns_per_row = coefficients_df_1.idxmax(axis=1)
        
        # Paragraph
        st.markdown("<br>" * 1, unsafe_allow_html=True)
        
        # centered output to display the correlations
        st.markdown("<div style='text-align: center;'><strong>Here are some Correlations with your current selection (likely to appear together):</strong></div>", unsafe_allow_html=True)   
        for col_name, value in max_columns_per_row.items():
            st.write(f"<div style='text-align: center;'>{col_name}: {value}</div>", unsafe_allow_html=True)
    
    except Exception as e:
        pass


# View 3 - Find a nearby Bar:    
    
elif selected_view == "Find a nearby Bar":
    
    #Setup Title
    st.title("Find a Bar nearby your Location 🌃")

    try: 
    
        # define function to retrieve coordinates for locations with an API and return latitude und longitude
        def get_coordinates(location):
            base_url = "https://nominatim.openstreetmap.org/search"
            params = {
                'q': location,
                'format': 'json',
                'limit': 1
            }

            response = requests.get(base_url, params=params)
            data = response.json()

            if data:
                first_result = data[0]
                lat, lon = float(first_result['lat']), float(first_result['lon'])
                return lat, lon
    
            else:
                st.markdown("<p style='text-align: center;'>Enter your location to start</p>", unsafe_allow_html=True)

        # use lat und lon to retrieve a list of bars within an individualised distance
        def search_nearby_bars(lat, lon, distance):
            base_url = "https://overpass-api.de/api/interpreter"
            query = f"""
                [out:json];
                node(around:{distance},{lat},{lon})["amenity"="pub"];
                out center;
            """

            response = requests.post(base_url, data=query)
            data = response.json()

            # Print the Results
            if 'elements' in data and data['elements']:
                for element in data['elements']:

                    st.markdown(f"<p style='text-align: center;'>{element['tags']['name']}</p>", unsafe_allow_html=True)

            else:
                st.markdown("<p style='text-align: center;'>No Bars nearby</p>", unsafe_allow_html=True)
         
        # Steup input fields for location and distance 
        location = st.text_input("Add your Location (Street, City, Country):", help = "e.g Brühlbleichestraße 8, St. Gallen, Switzerland")
        distance = st.slider('How far are you willing to go in meters?', min_value=1, max_value=5000, value=100)
    
        # Utilize the function to find lat und lon
        coords = get_coordinates(location)
        
        # Utilize the function to retrieve nearby bars
        if coords:
            st.markdown("<p style='text-align: center;'>Too lazy to shake cocktails yourself? <strong>Visit:</strong></p>", unsafe_allow_html=True)
            st.markdown("<br>" * 1, unsafe_allow_html=True)
            search_nearby_bars(*coords, distance)

    except Exception as e:
        pass
