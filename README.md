# A initial implementation of Mediator Pattern ported from .NET C# to Angular

## Interfaces and Mediator Service
### 1) IRequest
```
export interface IRequest<TResponse> { } 
```

### 2) IRequest Handler
```
import { IRequest } from "./irequest.interface";

export interface IRequestHandler<RequestType extends IRequest<TResponse>, TResponse>
{
    Handle(request: RequestType): TResponse;
}
```

### 3) Mediator Service
```
import { Injectable, Injector } from '@angular/core';
import { IRequestHandler } from './irequest-handler.interface';
import { IRequest } from './irequest.interface';

@Injectable({
    providedIn: 'root',
})
export class MediatorService {

    constructor(private inject: Injector) { }

    send(request: IRequest<Response>): Response 
    {
        let process = <IRequestHandler<IRequest<Response>, Response>>this.inject.get(request.constructor);
        let response = process.Handle(request);
        return response;
    }
}
```

### 4) Writing Commands, Queries and Handlers
**get-profile-by-id.query**
```
import { IRequest } from "src/app/shared/pattern/mediator";
import { ProfileModel } from "../models/profile.model";

export class GetProfileByIdQuery implements IRequest<ProfileModel> {
    id: number;

    constructor(id: number){
        this.id = id;
    }
}
```

**get-profile-by-id.query-handler**
```
import { HttpClient } from '@angular/common/http';
import { IRequestHandler } from "src/app/shared/pattern/mediator";
import { ProfileModel } from "../models/profile.model";
import { GetProfileByIdQuery } from "./get-profile-by-id.query";

export class GetProfileByIdQueryHandler implements IRequestHandler<GetProfileByIdQuery, ProfileModel>{

    Handle(request: GetProfileByIdQuery): ProfileModel {

        let response = new ProfileModel();
        response.id = request.id;
        response.name = 'Paulo ' + request.id.toString();
        return response;

    }    
}
```

### 5) Configuring Angular Mediator Service

You need to create a **mediator module** or use **some module** to configure the service.

```
import { HttpClient } from '@angular/common/http';
import { Injector, NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { GetProfileByIdQuery } from './profile/service/get-profile-by-id.query';
import { GetProfileByIdQueryHandler } from './profile/service/get-profile-by-id.query-handler';
import { MediatorService } from 'src/app/shared/pattern/mediator';

@NgModule({
  declarations: [],
  imports: [
    CommonModule
  ],
  providers: [
    //Mediator service added to use injcted in some component.
    { provide: MediatorService, useClass: MediatorService, deps: [ Injector ]},
  
    //Queries
    { provide: GetProfileByIdQuery, useClass: GetProfileByIdQueryHandler }

    //Commands

    //Notifications
    
  ]   
  
})
export class MediatorModule { }
```
