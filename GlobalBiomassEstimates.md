
# Global Biomass Outputs

Biomasses (in grams) are extracted using StateVariableGridTotal function:

```C#
public double StateVariableGridTotal(string variableName, string traitValue, int[] functionalGroups, List<uint[]> cellIndices, 
            string stateVariableType, MadingleyModelInitialisation initialisation)
        {

            double tempVal = 0;

            double[,] TempStateVariable = this.GetStateVariableGrid(variableName, traitValue, functionalGroups, cellIndices, stateVariableType, initialisation);

            // Loop through and sum values across a grid, excluding missing values
            for (int ii = 0; ii < cellIndices.Count; ii++)
            {
                tempVal += TempStateVariable[cellIndices[ii][0], cellIndices[ii][1]];
            }

            return tempVal;
        }
```

This sums the matrix that is returned by the GetStateVariableGrid function:

```C#
case "biomass":
                    for (int ii = 0; ii < cellIndices.Count; ii++)
                    {
                        // Check whether the state variable concerns cohorts or stocks
                        if (stateVariableType.ToLower() == "cohort")
                        {
                            if (traitValue != "Zooplankton")
                            {
                                // Check to make sure that the cell has at least one cohort
                                if (InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellCohorts != null)
                                {
                                    for (int nn = 0; nn < functionalGroups.Length; nn++)
                                    {
                                        if (InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellCohorts[functionalGroups[nn]] != null)
                                        {
                                            foreach (Cohort item in InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellCohorts[functionalGroups[nn]].ToArray())
                                            {
                                                    TempStateVariable[cellIndices[ii][0], cellIndices[ii][1]] += ((item.IndividualBodyMass + item.IndividualReproductivePotentialMass) * item.CohortAbundance);
                                            }
                                        }
                                    }
                                }
                            }
                            else
                            {
                                // Check to make sure that the cell has at least one cohort
                                if (InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellCohorts != null)
                                {
                                    for (int nn = 0; nn < functionalGroups.Length; nn++)
                                    {
                                        if (InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellCohorts[functionalGroups[nn]] != null)
                                        {
                                            foreach (Cohort item in InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellCohorts[functionalGroups[nn]].ToArray())
                                            {
                                                if (item.IndividualBodyMass <= initialisation.PlanktonDispersalThreshold)
                                                    TempStateVariable[cellIndices[ii][0], cellIndices[ii][1]] += ((item.IndividualBodyMass + item.IndividualReproductivePotentialMass) * item.CohortAbundance);
                                            }
                                        }
                                    }
                                }
                            }
                        }
                        else if (stateVariableType.ToLower() == "stock")
                        {
                            // Check to make sure that the cell has at least one stock
                            if (InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellStocks != null)
                            {
                                for (int nn = 0; nn < functionalGroups.Length; nn++)
                                {
                                    if (InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellStocks[functionalGroups[nn]] != null)
                                    {
                                        foreach (Stock item in InternalGrid[cellIndices[ii][0], cellIndices[ii][1]].GridCellStocks[functionalGroups[nn]].ToArray())
                                        {
                                            TempStateVariable[cellIndices[ii][0], cellIndices[ii][1]] += (item.TotalBiomass);

                                        }
                                    }

                                }
                            }
                        }
                        else
                        {
                            Debug.Fail("Variable 'state variable type' must be either 'stock' 'or 'cohort'");
                        }
                        
                    }
                    break;
```

In OutputGlobal.cs, these functions are called twice, once for cohorts and once for stocks:

```C#
TotalLivingBiomass += ecosystemModelGrid.StateVariableGridTotal("Biomass", "NA", CohortFunctionalGroupIndices, cellIndices, "cohort", initialisation);
TotalLivingBiomass += ecosystemModelGrid.StateVariableGridTotal("Biomass", "NA", StockFunctionalGroupIndices, cellIndices, "stock", initialisation);
```

In outputting to the console, total living biomass is reported in kg:
```C#
Console.WriteLine("Total biomass (all) = " + String.Format("{0:E}", TotalBiomass / 1000) + " kg");
```

In outputting to file, total living biomass is reported in grams:
```C#
DataConverter.ValueToSDS1D(TotalLivingBiomass, "Total living biomass", "Time step",
                ecosystemModelGrid.GlobalMissingValue, BasicOutput, (int)currentTimeStep + 1);
```

