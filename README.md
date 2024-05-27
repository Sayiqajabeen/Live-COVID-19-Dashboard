# Live-COVID-19-Dashboard
Gather relevant &amp; true information and display it on your dashboard
Fetching Real-Time COVID-19 Data
For real-time data, we can use APIs like the one provided by COVID19API.
import dash
import dash_core_components as dcc
import dash_html_components as html
import dash_table
import dash_bootstrap_components as dbc
from dash.dependencies import Input, Output
import pandas as pd
import requests

# Initialize the Dash app
app = dash.Dash(__name__, external_stylesheets=[dbc.themes.BOOTSTRAP])

# Function to fetch COVID-19 data
def fetch_data():
    url = "https://api.covid19api.com/summary"
    response = requests.get(url)
    data = response.json()
    return data

# Layout of the dashboard
app.layout = dbc.Container([
    dbc.Row([
        dbc.Col(html.H1("COVID-19 Live Dashboard"), className="mb-2")
    ]),
    dbc.Row([
        dbc.Col(html.H5("Global Statistics"), className="mb-4")
    ]),
    dbc.Row([
        dbc.Col(dbc.Card([
            dbc.CardBody([
                html.H4(id="total-cases", className="card-title"),
                html.P("Total Cases", className="card-text")
            ])
        ], color="primary", inverse=True), width=3),
        dbc.Col(dbc.Card([
            dbc.CardBody([
                html.H4(id="total-deaths", className="card-title"),
                html.P("Total Deaths", className="card-text")
            ])
        ], color="danger", inverse=True), width=3),
        dbc.Col(dbc.Card([
            dbc.CardBody([
                html.H4(id="total-recovered", className="card-title"),
                html.P("Total Recovered", className="card-text")
            ])
        ], color="success", inverse=True), width=3)
    ]),
    dbc.Row([
        dbc.Col(html.H5("Country Statistics"), className="mt-4 mb-4")
    ]),
    dbc.Row([
        dbc.Col(dcc.Dropdown(id="country-dropdown", multi=False, value="United States",
                             options=[], placeholder="Select a country"), width=6)
    ]),
    dbc.Row([
        dbc.Col(dash_table.DataTable(id="country-table", columns=[
            {"name": i, "id": i} for i in ["Country", "TotalConfirmed", "TotalDeaths", "TotalRecovered"]
        ], data=[], style_table={'overflowX': 'auto'}, style_cell={'textAlign': 'left'}), width=12)
    ])
])

# Callback to update global statistics
@app.callback(
    [Output("total-cases", "children"),
     Output("total-deaths", "children"),
     Output("total-recovered", "children")],
    [Input("country-dropdown", "value")]
)
def update_global_stats(_):
    data = fetch_data()
    global_stats = data["Global"]
    total_cases = f"{global_stats['TotalConfirmed']:,}"
    total_deaths = f"{global_stats['TotalDeaths']:,}"
    total_recovered = f"{global_stats['TotalRecovered']:,}"
    return total_cases, total_deaths, total_recovered

# Callback to update country dropdown and table
@app.callback(
    [Output("country-dropdown", "options"),
     Output("country-table", "data")],
    [Input("country-dropdown", "value")]
)
def update_country_data(selected_country):
    data = fetch_data()
    countries = data["Countries"]
    country_options = [{"label": country["Country"], "value": country["Country"]} for country in countries]
    country_data = [country for country in countries if country["Country"] == selected_country]
    return country_options, country_data

# Run the app
if __name__ == '__main__':
    app.run_server(debug=True)
#For running
python app.py



    
