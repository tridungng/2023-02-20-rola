# NGRX Selectors

- [NGRX Selectors](#ngrx-selectors)
  - [Adding a first selector](#adding-a-first-selector)
  - [Bonus: Using feature selectors \*](#bonus-using-feature-selectors-)
  - [Bonus: Using parameterized selectors \*](#bonus-using-parameterized-selectors-)
  - [Bonus: Compose complex component selector \*\*](#bonus-compose-complex-component-selector-)

## Adding a first selector

In this part of the lab, you'll add a selector that queries all the flights that are not on a defined negative list.

1. Open the file `flight-booking.reducer.ts` and add a property `negativeList` to your `State`:

   ```typescript
   export interface State {
     flights: Flight[];
     negativeList: number[];
   }

   export const initialState: State = {
     flights: [],
     negativeList: [3],
   };
   ```

   For the sake of simplicity, this example defines a default value for the negative list to filter the flight with the id 3.

2. In your `+state` folder, create a file `flight-booking.selectors.ts` and enter the following lines. If it already exists, update it as follows:

   ```typescript
   import { createSelector } from '@ngrx/store';
   import { FlightBookingAppState } from './flight-booking.reducer';

   export const selectFlights = (s: FlightBookingAppState) => s.flightBooking.flights;
   export const negativeList = (s: FlightBookingAppState) => s.flightBooking.negativeList;

   export const selectedFilteredFlights = createSelector(selectFlights, negativeList, (flights, negativeList) =>
     flights.filter((f) => !negativeList.includes(f.id))
   );
   ```

3. In your `flight-search.component.ts`, use the selector when fetching data from the store:

   ```typescript
   this.flights$ = this.store.select(selectedFilteredFlights);
   ```

4. Test your application.

## Bonus: Using feature selectors \*

To get rid of your FlightBookingAppState type, you can use a feature selector pointing to the branch of your feature:

```TypeScript
// Create feature selector
export const selectFlightBooking = createFeatureSelector<State>('flightBooking');

// Use feature selector to get data from feature branch
export const selectFlights = createSelector(selectFlightBooking, s => s.flights);

export const negativeList = createSelector(selectFlightBooking, s => s.negativeList);

[...]
```

## Bonus: Using parameterized selectors \*

You can pass a property object to a selector when calling it. This object is assigned to a further parameter in your selectors projection function.

1. In your `flight-booking.selectors.ts` file, add the following selector:

   ```typescript
   export const selectFlightsWithProps = (props: { blackList: number[] }) =>
     createSelector(selectFlights, (flights) => flights.filter((f) => !props.blackList.includes(f.id)));
   ```

   Please note that the projector get an additional `props` parameter. It points to a dynamic object.

2. Open the file `flight-search.component.ts` and fetch data with this selector:

   ```typescript
   this.flights$ = this.store.select(selectFlightsWithProps({ blackList: [3] }));
   ```

3. Test your solution.

## Bonus: Compose complex component selector \*\*

You use more complex selectors that reuse present selectors and compose a customized result that can be used in a concrete use case implemented in one of your smart components.

1. In your `flight-booking.reducer.ts` file, add the following state definition and initial state:

   ```typescript
   export interface State {
     flights: Flight[];
     // NEW:
     passenger: Record<
       number,
       {
         id: number;
         name: string;
         firstName: string;
       }
     >;
     bookings: {
       passengerId: number;
       flightId: number;
     }[];
     user: {
       name: string;
       passengerId: number;
     };
   }
   ```

   ```typescript
   export const initialState: State = {
     flights: [],
     // NEW:
     passenger: {
       1: { id: 1, name: 'Smith', firstName: 'Anne' },
     },
     bookings: [
       { passengerId: 1, flightId: 3 },
       { passengerId: 1, flightId: 5 },
     ],
     user: { name: 'anne.smith', passengerId: 1 },
   };
   ```

2. Open the file `flight-booking.selectors.ts` and implement all necessary selectors to select the new state properties.

   <details>
   <summary>Show code</summary>
   <p>

   ```typescript
   export const selectPassengers = createSelector(selectFlightBookingState, (state) => state.passenger);

   export const selectBookings = createSelector(selectFlightBookingState, (state) => state.bookings);

   export const selectUser = createSelector(selectFlightBookingState, (state) => state.user);
   ```

   </p>
   </details>

3. Define a new selector `selectActiveUserFlights` that returns only those flights that the active user has booked.

   <details>
   <summary>Show code</summary>
   <p>

   ```typescript
   export const selectActiveUserFlights = createSelector(
     // Selectors:
     selectFlights,
     selectBookings,
     selectUser,
     // Projector:
     (flights, bookings, user) => {
       const activeUserPassengerId = user.passengerId;
       const activeUserFlightIds = bookings
         .filter((b) => b.passengerId === activeUserPassengerId)
         .map((b) => b.flightId);
       const activeUserFlights = flights.filter((f) => activeUserFlightIds.includes(f.id));
       return activeUserFlights;
     }
   );
   ```

   </p>
   </details>

4. Try out your new selector `selectActiveUserFlights` by using it in the `flight-search.component.ts`.

5. Test your solution.