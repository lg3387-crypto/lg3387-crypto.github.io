[Porject 2.ipynb](https://github.com/user-attachments/files/23761729/Porject.2.ipynb)
{
 "cells": [
  {
   "cell_type": "markdown",
   "id": "a4adc5f9",
   "metadata": {},
   "source": [
    "*Author: Lingxin Guo \n",
    "*Course: Computing in Context – Project 2*  \n",
    "\n",
    "In this project, I explore the relationship between countries’ **use of renewable energy** and their **greenhouse gas (GHG) emissions per person**.  \n",
    "\n",
    "The guiding questions are:\n",
    "\n",
    "1. Are countries that rely more on renewable energy associated with **lower per-capita GHG emissions**?\n",
    "2. Are there visible regional patterns in this relationship?\n",
    "\n",
    "My hypothesis is that, *controlling loosely for development level*, countries with a **higher share of renewables** in their energy mix tend to have **lower GHG emissions per capita** – but there will be notable exceptions."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "d32733cf",
   "metadata": {},
   "source": [
    "## Data Sources\n",
    "\n",
    "For this project, I use **two separate datasets** from [Our World in Data](https://ourworldindata.org/):\n",
    "\n",
    "1. **Per-capita greenhouse gas emissions**  \n",
    "   - Data page: “Per capita greenhouse gas emissions” (Jones et al. 2024). :contentReference[oaicite:0]{index=0}  \n",
    "   - Downloaded via the grapher endpoint:  \n",
    "     `https://ourworldindata.org/grapher/per-capita-ghg-emissions.csv`  \n",
    "\n",
    "2. **Share of primary energy consumption from renewable sources**  \n",
    "   - Data page: “Share of primary energy consumption that comes from renewables”. :contentReference[oaicite:1]{index=1}  \n",
    "   - Downloaded via:  \n",
    "     `https://ourworldindata.org/grapher/renewable-share-energy.csv`  \n",
    "\n",
    "Both datasets are structured in the usual Our World in Data format: one row per **location–year** pair, with a value column for the indicator of interest. I treat them as two independent datasets and merge them using **country code** and **year**.\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "febb690a",
   "metadata": {},
   "source": [
    "import pandas as pd\n",
    "import matplotlib.pyplot as plt\n",
    "\n",
    "# Make plots a bit larger but still OK on an 11\" laptop\n",
    "plt.rcParams[\"figure.figsize\"] = (8, 6)\n",
    "\n",
    "ghg_url = \"https://ourworldindata.org/grapher/per-capita-ghg-emissions.csv\"\n",
    "renew_url = \"https://ourworldindata.org/grapher/renewable-share-energy.csv\"\n",
    "\n",
    "ghg_raw = pd.read_csv(ghg_url)\n",
    "renew_raw = pd.read_csv(renew_url)\n",
    "\n",
    "ghg_raw.head(), renew_raw.head()\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "54d12fcf",
   "metadata": {},
   "source": [
    "## Cleaning and Selecting Variables\n",
    "\n",
    "Both CSVs contain three key identifier columns:\n",
    "\n",
    "- `Entity` – country or world region name  \n",
    "- `Code` – 3-letter country code (blank for aggregates like “World”)  \n",
    "- `Year` – calendar year  \n",
    "\n",
    "Each file also includes one value column:\n",
    "\n",
    "- In the GHG dataset, the value is **per-capita GHG emissions** (tonnes of CO₂-equivalents per person).  \n",
    "- In the renewables dataset, the value is the **share of primary energy from renewables** (percent).\n",
    "\n",
    "To focus on comparable country-level data, I:\n",
    "\n",
    "1. **Drop aggregate regions** (rows without a 3-letter country code).  \n",
    "2. Keep only the columns I need.  \n",
    "3. Rename the value columns to descriptive variable names.\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "633a48fd",
   "metadata": {},
   "source": []
  },
  {
   "cell_type": "markdown",
   "id": "79aa97a3",
   "metadata": {},
   "source": [
    "# Inspect columns to be sure of names\n",
    "print(\"GHG columns:\", ghg_raw.columns.tolist())\n",
    "print(\"Renewables columns:\", renew_raw.columns.tolist())\n",
    "\n",
    "# Depending on the exact column names, adjust these:\n",
    "# Here I assume OWID's standard naming pattern.\n",
    "ghg = ghg_raw.rename(\n",
    "    columns={\n",
    "        \"Entity\": \"country\",\n",
    "        \"Code\": \"code\",\n",
    "        \"Year\": \"year\",\n",
    "        ghg_raw.columns[3]: \"ghg_per_capita\"  # value column\n",
    "    }\n",
    ")[[\"country\", \"code\", \"year\", \"ghg_per_capita\"]]\n",
    "\n",
    "renew = renew_raw.rename(\n",
    "    columns={\n",
    "        \"Entity\": \"country\",\n",
    "        \"Code\": \"code\",\n",
    "        \"Year\": \"year\",\n",
    "        renew_raw.columns[3]: \"renew_share\"  # value column\n",
    "    }\n",
    ")[[\"country\", \"code\", \"year\", \"renew_share\"]]\n",
    "\n",
    "# Drop aggregate regions (World, continents, etc.) – they have missing country code\n",
    "ghg = ghg.dropna(subset=[\"code\"])\n",
    "renew = renew.dropna(subset=[\"code\"])\n",
    "\n",
    "ghg.head(), renew.head()\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "8e649002",
   "metadata": {},
   "source": [
    "## Merging the Two Datasets\n",
    "\n",
    "To study a relationship, I need observations where **both** indicators are available in the **same year and country**.\n",
    "\n",
    "I merge the datasets on `code` and `year`, keeping only rows where both variables are non-missing. This gives me a panel dataset with:\n",
    "\n",
    "- `ghg_per_capita` – tonnes CO₂-eq per person  \n",
    "- `renew_share` – % of primary energy from renewables\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "15856867",
   "metadata": {},
   "source": [
    "merged = pd.merge(\n",
    "    ghg,\n",
    "    renew,\n",
    "    on=[\"code\", \"year\"],\n",
    "    suffixes=(\"_ghg\", \"_renew\")\n",
    ")\n",
    "\n",
    "# Drop rows with missing values in either column\n",
    "merged = merged.dropna(subset=[\"ghg_per_capita\", \"renew_share\"])\n",
    "\n",
    "print(\"Number of country-year observations:\", len(merged))\n",
    "merged.head()\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "9123a463",
   "metadata": {},
   "source": [
    "## Choosing a Year for the Main Visualization\n",
    "\n",
    "Our World in Data spans several decades. To make a **single, readable visualization**, I focus on **the most recent year** where many countries have data for both variables.\n",
    "\n",
    "I:\n",
    "\n",
    "1. Look at the distribution of years in the merged data.  \n",
    "2. Pick the latest year with a reasonably large number of countries.  \n",
    "3. Filter the dataset to that year for the main scatter plot.\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "05958075",
   "metadata": {},
   "source": [
    "# Check how many observations we have by year\n",
    "year_counts = merged[\"year\"].value_counts().sort_index(ascending=True)\n",
    "year_counts.tail(10)\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "1307b7b8",
   "metadata": {},
   "source": [
    "# Pick the latest year with at least, say, 80 countries\n",
    "candidate_years = year_counts[year_counts >= 80]\n",
    "latest_year = candidate_years.index.max()\n",
    "latest_year\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "a1090407",
   "metadata": {},
   "source": [
    "latest = merged[merged[\"year\"] == latest_year].copy()\n",
    "print(latest_year, latest.shape)\n",
    "latest.head()\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "aae4c669",
   "metadata": {},
   "source": [
    "## Visualization: Renewables vs. Per-Capita Emissions\n",
    "\n",
    "Now I create a **scatter plot** where each point is a country in the chosen year:\n",
    "\n",
    "- **X-axis:** Share of primary energy from renewables (`renew_share`, in %)  \n",
    "- **Y-axis:** Per-capita GHG emissions (`ghg_per_capita`, tonnes CO₂-eq per person)\n",
    "\n",
    "This single chart combines both datasets and lets us see whether higher use of renewables is associated with lower emissions per person.\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "d7e51bae",
   "metadata": {},
   "source": [
    "fig, ax = plt.subplots()\n",
    "\n",
    "ax.scatter(latest[\"renew_share\"], latest[\"ghg_per_capita\"], alpha=0.7)\n",
    "\n",
    "ax.set_title(\n",
    "    f\"Renewables vs. Per-Capita GHG Emissions, {latest_year}\"\n",
    ")\n",
    "ax.set_xlabel(\"Share of primary energy from renewables (%)\")\n",
    "ax.set_ylabel(\"Per-capita GHG emissions (tonnes CO₂-eq per person)\")\n",
    "\n",
    "# Optional: add a simple trend line (still computed from the data, no hard-coding)\n",
    "m, b = np.polyfit(latest[\"renew_share\"], latest[\"ghg_per_capita\"], 1)\n",
    "x_vals = pd.Series(sorted(latest[\"renew_share\"]))\n",
    "ax.plot(x_vals, m * x_vals + b)\n",
    "\n",
    "plt.tight_layout()\n",
    "plt.show()\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "327d45ce",
   "metadata": {},
   "source": [
    "import numpy as np\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "125c0946",
   "metadata": {},
   "source": [
    "## Interpreting the Relationship\n",
    "\n",
    "Visually, I look for:\n",
    "\n",
    "- Whether high-renewables countries tend to sit at **lower** levels of per-capita emissions.  \n",
    "- Whether there is a strong linear pattern or only a loose cloud of points.  \n",
    "- Outliers – countries with high renewables but still high emissions, or vice versa.\n",
    "\n",
    "To quantify the relationship, I also compute the **Pearson correlation coefficient** between the two variables.\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "fcfc456a",
   "metadata": {},
   "source": [
    "corr = latest[\"renew_share\"].corr(latest[\"ghg_per_capita\"])\n",
    "corr\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "7247b786",
   "metadata": {},
   "source": [
    "In my dataset for **{{latest_year}}**, the correlation between renewable share and per-capita GHG emissions is approximately **`<insert value from cell above>`**.\n",
    "\n",
    "- A **negative** value would suggest that, on average, countries with higher shares of renewables emit **less** per person.  \n",
    "- A correlation close to **zero** would mean the relationship is weak or noisy in this simple bivariate view.\n",
    "\n",
    "Looking at the scatter plot, I observe that:\n",
    "\n",
    "- \\<Write in your own words: e.g., “Many European countries cluster at moderate emissions and mid-to-high renewables, while some fossil-fuel-exporting countries show very high emissions and low renewables.”\\>  \n",
    "- \\<Comment on any clear outliers or regional patterns you notice.\\>\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "0a4f1864",
   "metadata": {},
   "source": [
    "## Limitations and Next Steps\n",
    "\n",
    "This quick analysis has several limitations:\n",
    "\n",
    "- **Correlation is not causation.** Countries with high renewables and low emissions might also differ in income level, industry structure, or climate policy – none of which I control for here.  \n",
    "- **Measurement differences.** GHG emissions include all sectors, while renewable share focuses on **primary energy**; the link between the two is indirect. :contentReference[oaicite:2]{index=2}  \n",
    "- **Data gaps.** Some countries are missing from one or both datasets in the chosen year, which may bias the sample toward data-rich countries.  \n",
    "- **No time dynamics.** Looking at only one year ignores how countries’ energy systems have evolved over time.\n",
    "\n",
    "If I had more time, I would:\n",
    "\n",
    "- Explore the **time series** dimension (e.g., how changes in renewables relate to changes in emissions).  \n",
    "- Group countries by **income level** or **region** to see whether the relationship differs for high-income vs. low-income countries.  \n",
    "- Add more variables, such as electricity mix or energy intensity, to better explain outliers.\n"
   ]
  }
 ],
 "metadata": {
  "language_info": {
   "name": "python"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 5
}
