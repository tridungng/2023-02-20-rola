# NGRX Entity

- [NGRX Entity](#ngrx-entity)
  - [Managing Passengers](#managing-passengers)
  - [Bonus: Loading passengers \*\*](#bonus-loading-passengers-)

## Managing Passengers

In this exercise, you will leverage `@ngrx/entity` and `@ngrx/schematics` to manage `Passenger` entities with the store. For this, you will create an separate PassengerModule with a PassengerComponent.

1. Use the CLI to generate a new `PassengersModule` with the boilerplate for `@ngrx/entity`. For this, switch into the folder `flight-app\src\app` and use the following commands:

   ```
   npx ng g module passengers
   cd passengers
   npx ng g @ngrx/schematics:entity Passenger --module passengers.module.ts --creators
   ```

2. Discover the generated files.

3. Open the file `passenger.model` and add a `name` property to the `Passenger` class. Also, **make the id a number**:

   ```TypeScript
   export interface Passenger {
       id: number;    // <-- Modify (number)
       name: string;  // <-- Add this
   }
   ```

4. In the `passengers` folder, create a new file `passenger.selectors.ts`:

   ```typescript
   import * as fromPassenger from './passenger.reducer';
   import { createSelector } from '@ngrx/store';
   import { passengersFeatureKey } from './passenger.reducer';

   // Parent node pointing to passenger state
   export class PassengerAppState {
     [passengersFeatureKey]: fromPassenger.State;
   }

   // Selector pointing to passenger state in store
   const base = (s: PassengerAppState) => s.passengers;

   // Selector pointing to all passenger entities
   export const selectAllPassengers = createSelector(base, fromPassenger.selectAll);
   ```

5. In the `passengers` folder, create a new `PassengersComponent`. In its `ngOnInit` method, send an `AddPassengers` action with an hard coded array of passengers to the store and query all the passengers using the above mentioned `selectAllPassengers` selector. Display the passengers in the template.

   <details>
   <summary>Show code (TypeScript)</summary>
   <p>

   ```TypeScript
   @Component({
       selector: 'app-passengers',
       templateUrl: './passengers.component.html',
       styleUrls: ['./passengers.component.css']
   })
   export class PassengersComponent implements OnInit {

       constructor(private store: Store<PassengerAppState>) {}

       passengers$: Observable<Passenger[]>;

       ngOnInit(): void {
           this.store.dispatch(addPassengers({ passengers: [{id: 1, name: 'Max'}, {id:2, name: 'Susi'}]}));
           this.passengers$ = this.store.select(selectAllPassengers);
       }

   }
   ```

   </p>
   </details>

   <details>
   <summary>Show code (HTML)</summary>
   <p>

   ```html
   <div class="card">
     <div class="header">
       <h2 class="title">Latest Passengers</h2>
     </div>
     <div class="content">
       <pre>{{ passengers$ | async | json}}</pre>
     </div>
   </div>
   ```

   </p>
   </details>

6. Make sure, the `PassengersComponent` is declared **AND** exported with the `PassengerModule`.

7. Make sure, the `PassengersModule` is imported into the `AppModule`.

8. Call the `PassengersComponent` within the `HomeComponent` to try it out.

   ```html
   <app-passengers></app-passengers>
   ```

9. Test your application.

## Bonus: Loading passengers \*\*

Extend your solution to load passengers using a search form and an effect. You can use the following Web API for this:

    http://angular.at/api/passenger?name=Muster

Please note that this Web API is using PascalCase to display attributes with XML but camelCase for JSON to respect the respective usual conventions.
