# NestJS Commander

Have you been building amazing REST and RPC applications with [NestJS](https://docs.nestjs.com/)? Do you want that same structure for absolutely everything you're working with? Have you always wanted to build up some sweet CLI application but don't really know where to start? This is the solution. A package to bring building CLI applications to the Nest world with the same structure that you already know and love :heart: Built on top of the popular [Commander](https://github.com/tj/commander.js) package. 

## Installation

Before you get started, you'll need to install a few packages. First and foremost, this one: `nestjs-commander` (name pending). You'll also need to install `@nestjs/common` and `@nestjs/core` as this package makes use of them under the hood, but doesn't want to tie you down to a specific version, yay peerDependencies!

```sh
npm i nestjs-commander @nestjs/common @nestjs/core
# OR
yarn add nestjs-commander @nestjs/common @nestjs/core
# OR
pnpm i nestjs-commander @nestjs/common @nestjs/core
```

## A Command File

`nestjs-commander` makes it easy to write new command line applications with [decorators](https://www.typescriptlang.org/docs/handbook/decorators.html) via the `@Command()` decorator for classes and the `@Option()` decorator for methods of that class. Every command file _should_ implement the `CommandRunner` interface and _should_ be decorated with a `@Command()` decorator.

### CommandRunner

Every command is seen as an `@Injectable()` by Nest, so your normal Dependency Injection still works as you would expect it to (woohoo!). The only thing to take note of is the interface `CommandRunner`, which should be implemented by each command. The `CommandRunner` interface ensures that all commands have a `run` method that return a `Promise<void>` and takes in the parameters `string[], Record<string, any>`. The `run` command is where you can kick all of your logic off from, it will take in whatever parameters did not match option flags and pass them in as an array, just in case you are really meaning to work with multiple parameters. As for the options, the `Record<string, any>`, the names of these properties match the `name` property given to the `@Option()` decorators, while their value matches the return of the option handler. If you'd like better type safety, you are welcome to create an interface for your options as well. You can view how the [Basic Command test](test/basic.command.ts) manages that if interested.

### @Command()

The `@Command()` decorator is to define what CLI command the class is going to manage and take care of. The decorator takes in an object to define properties of the command.  The options passed here would be the same as the options passed to a new `command` for Commander

| property | type | required | description | 
| - | - | - | - |
| nameAndArgs | string | true | the name of the command and args it can tak. Args are optional if you decide to use the decorator approach with `@Option()` |
| description | string | false | the description of the command. This will be used by the `--help` or `-h` flags to have a formalized way of what to print out |
| Options | CommandOptions | false | Extra options to pass on down to commander |

### @Option()

Often times you're not just running a single command with a single input, but rather you're running a command with multiple options and flags. Think of something like `git commit`: you can pass a `--amend` flag, a `-m` flag, or even `-a`, all of these change how the command runs. These flags are able to be set for each command using the `@Option()` decorator on a method for how that flag should be parsed. Do note that every command sent in via the command line is a raw string, so if you need to transform that string to a number or a boolean, or any other type, this handler is where it can be done. See the [putting it all together](#putting-it-all-together) for an example. The `@Option()` decorator, like the `@Command()` one, takes in an object of options defined in the table below

| property | type | required | description |
| - | - | - | - |
| flags | string | true | a string that represents the option's incoming flag and if the option is required (using <>) or optional (using []) |
| description | string | false | the description of the option, used if adding a `--help` flag |
| defaultValue | string or boolean | false | the default value for the flag |

Under the hood, the method that the`@Option()` is decorating is the custom parser passed to commander for how the value should be parsed. This means if you want to parse a boolean value, the best way to do so would be to use `JSON.parse(val)` as `Boolean('false')` actually returns `true` instead of the expected `false`.

## Running the Command

Similar to how in a NestJS application we can use the `NestFactory` to create a server for us, and run it using `listen`, the `nestjs-commander` package exposes a simple to use API to run your server. Import the `CommandFactory` and use the `static` method `run` and pass in the root module of your application. This would probably look like below

```ts
import { CommandFactory } from 'nestjs-commander';
import { AppModule } from './app.module';

async function bootstrap() {
  await CommandFactory.run(AppModule);
}

bootstrap();
```

And that's it. Under the hood, `CommandFactory` will worry about calling `NestFactory` for you and calling `app.close()` when necessary, so you shouldn't need to worry about memory leaks there. If you need to add in some error handling, there's always `try/catch` wrapping the `run` command, or you can chain on some `.catch()` method to the `bootstrap()` call.

## Putting it All Together

I will make this section soon, I promise. 