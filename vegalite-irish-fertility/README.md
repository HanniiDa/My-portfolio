# Irish Fertility Trends — Vega-Lite Visualization

Interactive visualization exploring fertility trends in Ireland by age group and region, built with Vega-Lite as part of my MSc in Social Data Science at UCD.

## What this shows
- Comparison of Ireland's total fertility rate (TFR) against other European countries from 1970 to 2024
- Age-specific fertility rates by NUTS 3 region, broken down by childbearing age group (early, typical, late)
- Trends over time across all regions, with an interactive toggle between "all ages" and "by childbearing age group" views

The visualization highlights how Ireland's fertility patterns have shifted over five decades, both in international comparison and across regions, with a particular focus on the shift toward later childbearing.

## Tools
Vega-Lite, Google Sheets (data source)

---

```vega-lite
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "title": {
    "text": "Fertility Rate in Ireland by Age Group and Region",
    "dx": -70,
    "offset": 20,
    "fontSize": 25,
    "anchor": "middle"
  },
  "params": [
    {
      "name": "View",
      "value": "Total",
      "bind": {
        "input": "select",
        "options": ["Total", "By age group"],
        "labels": ["All ages (15–49 yo)", "By Childbearing age group"]
      }
    }
  ],
  "vconcat": [
    {
      "hconcat": [
        {
          "width": 150,
          "height": 300,
          "data": {
            "url": "https://docs.google.com/spreadsheets/d/e/2PACX-1vTZxcOWvhzWntqNtXPcj2p2kmxlxmRWrmqwOiJfh8ph2Lvk__g1Ku7aLOuBx1QsGWxjEW4Z7b3SXnu7/pub?gid=1079030144&single=true&output=csv",
            "format": {"type": "csv"}
          },
          "title": {"text": "Fertility Rate in Europe", "fontSize": 10},
          "encoding": {
            "y": {
              "field": "TFR",
              "type": "quantitative",
              "scale": {"zero": false},
              "axis": {
                "title": "",
                "labels": false,
                "ticks": false,
                "grid": false
              }
            },
            "x": {
              "field": "Period",
              "type": "ordinal",
              "sort": ["1970", "2024"],
              "axis": {"title": "", "labelAngle": 0}
            },
            "detail": {"field": "Country"},
            "color": {
              "condition": {
                "test": "datum.Country === 'Ireland'",
                "value": "#7a0177"
              },
              "value": "lightgray"
            },
            "opacity": {
              "condition": {"test": "datum.Country === 'Ireland'", "value": 1},
              "value": 0.6
            }
          },
          "layer": [
            {"mark": {"type": "line", "strokeWidth": 2, "point": true}},
            {
              "transform": [{"filter": "datum.Period == 1970"}],
              "mark": {
                "type": "text",
                "align": "right",
                "dx": -4,
                "baseline": "middle",
                "fontSize": 10,
                "fontWeight": "bold"
              },
              "encoding": {
                "text": {"field": "Country"},
                "tooltip": [
                  {"field": "Country", "type": "nominal"},
                  {
                    "field": "TFR",
                    "type": "quantitative",
                    "title": "fertility rate"
                  }
                ]
              }
            },
            {
              "transform": [{"filter": "datum.Period == 2024"}],
              "mark": {
                "type": "text",
                "align": "left",
                "dx": 4,
                "baseline": "middle",
                "fontSize": 10,
                "fontWeight": "bold"
              },
              "encoding": {
                "text": {"field": "Country"},
                "tooltip": [
                  {"field": "Country", "type": "nominal"},
                  {
                    "field": "TFR",
                    "type": "quantitative",
                    "title": "fertility rate"
                  }
                ]
              }
            }
          ]
        },
        {
          "data": {
            "url": "https://docs.google.com/spreadsheets/d/e/2PACX-1vRW6dhm_YWOM9__XvmZEoF3cMr7iVuT3bpnjRdzX6BPRuj0Zg-UlUlfCESKCyMYd9M-Esxg5MEMbjcO/pub?gid=97148953&single=true&output=csv",
            "format": {"type": "csv"}
          },
          "transform": [
            {"filter": "datum['NUTS 3 Region'] !== 'Ireland'"},
            {"filter": "datum['Age Group'] !== 'All ages' "},
            {
              "calculate": "View == 'By age group' ? (datum['Age Group'] == '15 - 19 years' || datum['Age Group'] == '20 - 24 years' ? 'Early childbearing (15-24 yo)' : (datum['Age Group'] == '25 - 29 years' || datum['Age Group'] == '30 - 34 years' ? 'Typical childbearing (25-34 yo)' : (datum['Age Group'] == '35 - 39 years' || datum['Age Group'] == '40 - 44 years' || datum['Age Group'] == '45 - 49 years' ? 'Late childbearing (35-49 yo)' : 'Other'))) : replace(datum['Age Group'], ' years', '')",
              "as": "childbearingAge"
            },
            {"filter": {"selection": "ageSelect"}}
          ],
          "selection": {
            "ageSelect": {
              "type": "multi",
              "fields": ["childbearingAge"],
              "bind": "legend"
            }
          },
          "width": 500,
          "height": 300,
          "mark": "bar",
          "encoding": {
            "x": {
              "field": "NUTS 3 Region",
              "type": "nominal",
              "sort": {"op": "mean", "field": "VALUE", "order": "descending"},
              "title": "Region",
              "axis": {
                "labelFontSize": 10,
                "titleFontSize": 10,
                "labelAngle": 0
              }
            },
            "y": {
              "aggregate": "mean",
              "field": "VALUE",
              "type": "quantitative",
              "title": "Sum of age-specific fertility rates (per 1,000 women)"
            },
            "color": {
              "field": "childbearingAge",
              "type": "nominal",
              "title": "Age group / Childbearing group",
              "scale": {
                "domain": [
                  "15 - 19",
                  "20 - 24",
                  "25 - 29",
                  "30 - 34",
                  "35 - 39",
                  "40 - 44",
                  "45 - 49",
                  "───────────",
                  "Early childbearing (15-24 yo)",
                  "Typical childbearing (25-34 yo)",
                  "Late childbearing (35-49 yo)"
                ],
                "range": [
                  "#feebe2",
                  "#fcc5c0",
                  "#fa9fb5",
                  "#f768a1",
                  "#dd3497",
                  "#ae017e",
                  "#7a0177",
                  "#FFFFFF",
                  "#fcc5c0",
                  "#f768a1",
                  "#7a0177"
                ]
              },
              "legend": {"labelLimit": 0, "symbolSize": 100}
            },
            "order": {"field": "childbearingAge", "type": "nominal"}
          }
        }
      ]
    },
    {
      "data": {
        "url": "https://docs.google.com/spreadsheets/d/e/2PACX-1vRW6dhm_YWOM9__XvmZEoF3cMr7iVuT3bpnjRdzX6BPRuj0Zg-UlUlfCESKCyMYd9M-Esxg5MEMbjcO/pub?gid=97148953&single=true&output=csv",
        "format": {"type": "csv"}
      },
      "transform": [
        {"filter": "datum['Age Group'] !== 'All ages' "},
        {
          "calculate": "View == 'By age group' ? (datum['Age Group'] == '15 - 19 years' || datum['Age Group'] == '20 - 24 years' ? 'Early childbearing (15-24 yo)' : (datum['Age Group'] == '25 - 29 years' || datum['Age Group'] == '30 - 34 years' ? 'Typical childbearing (25-34 yo)' : (datum['Age Group'] == '35 - 39 years' || datum['Age Group'] == '40 - 44 years' || datum['Age Group'] == '45 - 49 years' ? 'Late childbearing (35-49 yo)' : 'Other'))) : replace(datum['Age Group'], ' years', '')",
          "as": "childbearingAge"
        },
        {"filter": {"selection": "ageSelect"}}
      ],
      "width": 800,
      "height": 300,
      "mark": {"type": "line", "point": true, "strokeWidth": 2},
      "encoding": {
        "x": {
          "field": "Year",
          "type": "ordinal",
          "title": "Year",
          "axis": {
            "labelFontSize": 10,
            "labelAngle": 0,
            "values": [
              "1970","1972","1974","1976","1978","1980","1982","1984","1986","1988",
              "1990","1992","1994","1996","1998","2000","2002","2004","2006","2008",
              "2010","2012","2014","2016","2018","2020","2022","2024"
            ]
          }
        },
        "y": {
          "aggregate": "mean",
          "field": "VALUE",
          "type": "quantitative",
          "axis": {
            "title": "Age-specific fertility rates (per 1,000 women)",
            "grid": true
          }
        },
        "color": {
          "field": "childbearingAge",
          "type": "nominal",
          "scale": {
            "domain": [
              "15 - 19","20 - 24","25 - 29","30 - 34","35 - 39","40 - 44","45 - 49",
              "───────────",
              "Early childbearing (15-24 yo)","Typical childbearing (25-34 yo)","Late childbearing (35-49 yo)"
            ],
            "range": [
              "#feebe2","#fcc5c0","#fa9fb5","#f768a1","#dd3497","#ae017e","#7a0177",
              "#FFFFFF",
              "#fcc5c0","#f768a1","#7a0177"
            ]
          },
          "title": "Age group / Childbearing group",
          "legend": {"labelLimit": 0}
        },
        "tooltip": [
          {"field": "Year", "type": "ordinal", "title": "Year"},
          {"field": "childbearingAge", "type": "nominal", "title": "Age Group"},
          {
            "aggregate": "mean",
            "field": "VALUE",
            "type": "quantitative",
            "title": "ASFR"
          }
        ]
      }
    }
  ],
  "config": {}
}
```
## What this shows
- Comparison of Ireland's total fertility rate (TFR) against other European countries from 1970 to 2024
- Age-specific fertility rates by NUTS 3 region, broken down by childbearing age group (early, typical, late)
- Trends over time across all regions, with an interactive toggle between "all ages" and "by childbearing age group" views

## Objective of the visualization 
The visualization highlights how Ireland's fertility patterns have shifted over five decades, both in international comparison and across regions, with a particular focus on the shift toward later childbearing.

## Tools
Vega-Lite, Google Sheets (data source)
