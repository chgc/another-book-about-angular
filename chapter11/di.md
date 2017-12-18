# Angular Dependency Injection

Angular 有內建一套 Dependency Injection 框架，可以讓我們簡單的完成 DI 的相關行為，以下簡單說明如何註冊並使用

# 註冊 providers 

我們可以在 `NgModule` 與 `Component` 這兩個地方註冊 providers，

```typescript
@NgModule({
  imports: [
    BrowserModule
  ],
  declarations: [
   ...
  ],
  providers: [
    UserService, // 註冊 provider
    { provide: APP_CONFIG, useValue: HERO_DI_CONFIG }
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```

```typescript
import { Component }          from '@angular/core';

import { HeroService }        from './hero.service';

@Component({
  selector: 'my-heroes',
  providers: [HeroService], // 註冊 provider
  ...
})
export class HeroesComponent { }
```

而註冊這這兩個地方的差異性是影響的層級，如果 provider 是註冊在 `NgModule` 時，是註冊在 Application 層級或是 Module 層級，在該層級下的所有人都可以使用到同一個 provider，註冊在 `Component` 時，只能在該 component 及向下的子 component 可以使用而已。所以可依據該 provider 的使用群組來決定要註冊在哪一個地方

註冊 provider 的方式有很多種，但是在這之前，必須提到 `@Injectable` 這個 decorator，任何需要被註冊到 provider 裡的 service 都必須掛有 `@Injectable` ，不然 Injector 會報錯誤，雖然這表示不需註冊至 provider 內的 service 不需要標示為 `@Injectable`，但官網建議為了一致性，還是將其 decorator 標上。



## 註冊方法

Angular Provider 的註冊方式有以下幾種

1. shorthand 表示法：直接將 service 註冊到 providers 內

   ```typescript
   providers: [Logger]
   ```

   這種註冊式方式等同於下一個的寫法

2. useClass : 會建立一個新的 instance

   ```typescript
   providers:[
       {provide: Logger, useClass: Logger}
   ]
   ```

3. useExist：使用已存在的 provider，有使用別名的效果

   ```typescript
   providers:[
       Logger,
       {provide: myLogger, useExist: Logger}
   ]
   ```

4. useValue：單純的想提供值而非一個 class 時，就可以使用 useValue 的方式註冊，建議搭配 InjectionToken 一起使用

   ```typescript
   import { InjectionToken } from '@angular/core';

   export let APP_CONFIG = new InjectionToken<AppConfig>('app.config');
   ...
   providers: [{ provide: APP_CONFIG, useValue: HERO_DI_CONFIG }]
   ```

5. useFactory：利用工廠模式來建立 Class 實體

   ```typescript
   let heroServiceFactory = (logger: Logger, userService: UserService) => {
     return new HeroService(logger, userService.user.isAuthorized);
   };
   ...
   providers:[
       { provide: HeroService,
         useFactory: heroServiceFactory,
         deps: [Logger, UserService]
       }
   ]  
   ```

   **deps** 提供 FacotryClass 所需的 dependence class



# 使用 providers

當完成註冊 providers 後，要如何使用已註冊的 provider 呢? 以 component 為例，將要使用的 provider 列在 constructor 內，Angular 在 compiler 時就會進行分析並將設定的 provider 提供給 Component 使用

可以透過以下的三種方式

1. 直接使用 provider type 

```typescript
export class HeroListComponent {
  heroes: Hero[];
 
  constructor(heroService: HeroService) {
    this.heroes = heroService.getHeroes();
  }
}
```

2. 使用 Injector

```typescript
  constructor(private injector: Injector) {
    heroService = this.injector.get(HeroService);
    this.heroes = heroService.getHeroes();
  }
```

3. `@Inject`

註冊 providers 時，有時候會使用 `useValue` 的方式註冊 Value 至 provider 內，這時需透過 `@Inject()` 的方式取得所註冊的值

```typescript
import { InjectionToken } from '@angular/core';

export let APP_CONFIG = new InjectionToken<AppConfig>('app.config');
providers: [{ provide: APP_CONFIG, useValue: HERO_DI_CONFIG }]

// component
constructor(@Inject(APP_CONFIG) config: AppConfig) {
  this.title = config.title;
}
```



# 設定 Injector

當啟動 module 時，Angular 就會幫我們建立一個 Application 層級的 Injector

```typescript
platformBrowserDynamic().bootstrapModule(AppModule);
```

## 底層運作方式 (純參考)

從 `platformBrowserDynamic()` 執行時，就會開始一系列的初始建置 platform 的動作，這其中包含 Injector 的設定，從原始碼整理出的順序是執行 `platformBrowerDynamic()` 事實上就是執行 `createPlatformFactory`的動作。

```typescript
// 進入點
export const platformBrowserDynamic = createPlatformFactory(
    platformCoreDynamic, 'browserDynamic', INTERNAL_BROWSER_DYNAMIC_PLATFORM_PROVIDERS);

// 第二層的 platformCoreDynamic
export const platformCoreDynamic = createPlatformFactory(platformCore, 'coreDynamic', [
  {provide: COMPILER_OPTIONS, useValue: {}, multi: true},
  {provide: CompilerFactory, useClass: JitCompilerFactory, deps: [COMPILER_OPTIONS]},
]);

// 最底層的 platformCore
export const platformCore = createPlatformFactory(null, 'core', _CORE_PLATFORM_PROVIDERS);
```

`createPlatformFactory` 接受三個參數，

1. parent platform: 這裡就是 `platformCore`
2. platform 的名稱
3. 要註冊至 Injector 的 provider ，型別為 `StaticProvider`

在 `createPlatFormFactory` 裡，如果沒有 parent platform 時，才會建立 Injector，而這裡就是指 platformCore 的部分，也是我們要關注的地方， **Injector.Create**

```typescript
export function createPlatformFactory(
    parentPlatformFactory: ((extraProviders?: StaticProvider[]) => PlatformRef) | null,
    name: string, providers: StaticProvider[] = []): (extraProviders?: StaticProvider[]) =>
    PlatformRef {
  const marker = new InjectionToken(`Platform: ${name}`);
  return (extraProviders: StaticProvider[] = []) => {
    let platform = getPlatform();
    if (!platform || platform.injector.get(ALLOW_MULTIPLE_PLATFORMS, false)) {
      if (parentPlatformFactory) {
        parentPlatformFactory(
            providers.concat(extraProviders).concat({provide: marker, useValue: true}));
      } else {
        // 建立 Injector
        createPlatform(Injector.create(
            providers.concat(extraProviders).concat({provide: marker, useValue: true})));
      }
    }
    return assertPlatform(marker);
  };
}
```

`Injector` 裡的 create 方法是一個 static method，真正使用到 Injector.create 的地方有三處，其餘的皆測試檔案裡使用。

1. createPlatformFactory
2. compiler_factory
3. TestBed

```typescript
export abstract class Injector {
  static THROW_IF_NOT_FOUND = _THROW_IF_NOT_FOUND;
  static NULL: Injector = new _NullInjector();
  ... 
  static create(providers: StaticProvider[], parent?: Injector): Injector {
    return new StaticInjector(providers, parent);
  }
}
```

`StaticInjector` 的程式碼如下

```typescript
export class StaticInjector implements Injector {
  readonly parent: Injector;

  private _records: Map<any, Record>;

  constructor(providers: StaticProvider[], parent: Injector = NULL_INJECTOR) {
    this.parent = parent;    
    // 重點在這裡
    const records = this._records = new Map<any, Record>();
    records.set(
        Injector, <Record>{token: Injector, fn: IDENT, deps: EMPTY, value: this, useNew: false});
    recursivelyProcessProviders(records, providers);
  }

  get<T>(token: Type<T>|InjectionToken<T>, notFoundValue?: T): T;
  get(token: any, notFoundValue?: any): any;
  get(token: any, notFoundValue?: any): any {
   ...
  }

  toString() {
  ...
  }
}
```

由此可以看出 Injector 其實內部是透過 Map 的方式來管理 Providers 的，而 `recursivelyProcessProviders` 是用來將傳入的 providers 陣列註冊到 records 中，以供後續使用。

```typescript
function recursivelyProcessProviders(records: Map<any, Record>, provider: StaticProvider) {
  if (provider) {
    provider = resolveForwardRef(provider);
    if (provider instanceof Array) {
      // if we have an array recurse into the array
      for (let i = 0; i < provider.length; i++) {
        recursivelyProcessProviders(records, provider[i]);
      }
    } else if (typeof provider === 'function') {     
      throw staticError('Function/Class not supported', provider);
    } else if (provider && typeof provider === 'object' && provider.provide) {      
      // 對應 forwardRef decorator
      let token = resolveForwardRef(provider.provide);
      // resolveProvider 是另外一個重點 function，解析各式的 provider 設定
      const resolvedProvider = resolveProvider(provider);
      if (provider.multi === true) {
        // This is a multi provider.
        let multiProvider: Record|undefined = records.get(token);
        if (multiProvider) {
          if (multiProvider.fn !== MULTI_PROVIDER_FN) {
            throw multiProviderMixError(token);
          }
        } else {
          // Create a placeholder factory which will look up the constituents of the multi provider.
          records.set(token, multiProvider = <Record>{
            token: provider.provide,
            deps: [],
            useNew: false,
            fn: MULTI_PROVIDER_FN,
            value: EMPTY
          });
        }
        // Treat the provider as the token.
        token = provider;
        multiProvider.deps.push({token, options: OptionFlags.Default});
      }
      const record = records.get(token);
      if (record && record.fn == MULTI_PROVIDER_FN) {
        throw multiProviderMixError(token);
      }
      records.set(token, resolvedProvider);
    } else {
      throw staticError('Unexpected provider', provider);
    }
  }
}
```

在上列的程式碼中，有幾個重點

1. forwardRef：Allows to refer to references which are not yet defined.

   ```typescript
   export function resolveForwardRef(type: any): any {
     if (typeof type === 'function' && type.hasOwnProperty('__forward_ref__') &&
         type.__forward_ref__ === forwardRef) {
       return (<ForwardRefFn>type)();
     } else {
       return type;
     }
   }
   ```

2. resolveProvider：分析 provider 的註冊是否合法

   ```typescript
   function resolveProvider(provider: SupportedProvider): Record {
     const deps = computeDeps(provider);
     let fn: Function = IDENT;
     let value: any = EMPTY;
     let useNew: boolean = false;
     let provide = resolveForwardRef(provider.provide);
     if (USE_VALUE in provider) {
       // We need to use USE_VALUE in provider since provider.useValue could be defined as undefined.
       value = (provider as ValueProvider).useValue;
     } else if ((provider as FactoryProvider).useFactory) {
       fn = (provider as FactoryProvider).useFactory;
     } else if ((provider as ExistingProvider).useExisting) {
       // Just use IDENT
     } else if ((provider as StaticClassProvider).useClass) {
       useNew = true;
       fn = resolveForwardRef((provider as StaticClassProvider).useClass);
     } else if (typeof provide == 'function') {
       useNew = true;
       fn = provide;
     } else {
       throw staticError(
           'StaticProvider does not have [useValue|useFactory|useExisting|useClass] or [provide] is not newable',
           provider);
     }
     return {deps, fn, useNew, value};
   }
   ```

到這裡，已經可以看出 Inector 的運作模式，但到這個階段也同時建立 Application-Wild 層級的 Injector，預設在 platform 層級所註冊的 provider 有下列幾項

1. {provide: ResourceLoader, useClass: CachedResourceLoader, deps: []}
2. {provide: COMPILER_OPTIONS, useValue: {}, multi: true}
3. **{provide: CompilerFactory, useClass: JitCompilerFactory, deps: [COMPILER_OPTIONS]}**
4. {provide: PLATFORM_ID, useValue: 'unknown'}
5. **{provide: PlatformRef, deps: [Injector]}** 
6. {provide: TestabilityRegistry, deps: []}
7. {provide: Console, deps: []}

之前有提過 Injector.create 除了測試檔案外，只有三個地方會出現，而 `CompilerFactory` 是其中一個，這裡我們可以知道 `CompilerFactory` 是使用 `JitCompilerFactory` 來編譯 Component，這裡另外建立一個 Injector，所以這裡所註冊的 Provider 會限制於在此層級使用

```typescript
export class JitCompilerFactory implements CompilerFactory {
  private _defaultOptions: CompilerOptions[];
  constructor(defaultOptions: CompilerOptions[]) {
    const compilerOptions: CompilerOptions = {
      useJit: true,
      defaultEncapsulation: ViewEncapsulation.Emulated,
      missingTranslation: MissingTranslationStrategy.Warning,
      enableLegacyTemplate: false,
    };

    this._defaultOptions = [compilerOptions, ...defaultOptions];
  }
  createCompiler(options: CompilerOptions[] = []): Compiler {
    const opts = _mergeOptions(this._defaultOptions.concat(options));   
    const injector = Injector.create([
      COMPILER_PROVIDERS, {
        provide: CompilerConfig,
        useFactory: () => {
          return new CompilerConfig({
            // let explicit values from the compiler options overwrite options
            // from the app providers
            useJit: opts.useJit,
            jitDevMode: isDevMode(),
            // let explicit values from the compiler options overwrite options
            // from the app providers
            defaultEncapsulation: opts.defaultEncapsulation,
            missingTranslation: opts.missingTranslation,
            enableLegacyTemplate: opts.enableLegacyTemplate,
            preserveWhitespaces: opts.preserveWhitespaces,
          });
        },
        deps: []
      },
      opts.providers !
    ]);
    return injector.get(Compiler);
  }
}
```

在 `JitCompilerFactory` 內的 `createCompiler` 函式會在 `bootstrapModule` 時被執行，`bootstrapModule` 是 `platformBrowserDynamic()` 建立後的下一個動作，`bootstrapModule` 定義在 `PlatformRef` 中

```typescript
platformBrowserDynamic().bootstrapModule(AppModule);
```

```typescript
  bootstrapModule<M>(
      moduleType: Type<M>, compilerOptions: (CompilerOptions&BootstrapOptions)|
      Array<CompilerOptions&BootstrapOptions> = []): Promise<NgModuleRef<M>> {
    const compilerFactory: CompilerFactory = this.injector.get(CompilerFactory);
    const options = optionsReducer({}, compilerOptions);
    // 建立另外一個新的 Injector
    const compiler = compilerFactory.createCompiler([options]);

    return compiler.compileModuleAsync(moduleType)
        .then((moduleFactory) => this.bootstrapModuleFactory(moduleFactory, options));
  }
```

以上就是 Angular 建置 Injector 的基本流程

