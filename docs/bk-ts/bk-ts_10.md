# 第十章：泛型

# 泛型

TypeScript 支持一种称为 *generics* 的编程结构。TypeScript 泛型允许您编写针对广泛类和接口的代码，而不放弃强类型。您编写代码以针对 *类型*（类和接口），而不是具体的、明确的类。一旦编写完成，您通过在运行时提供具体类型来访问这个通用代码。让我们考虑一个例子。

想象一下，您正在开发一个游戏，并将游戏信息存储到/从数据库中检索。这意味着您必须为游戏中的各种对象实现经典的创建、读取、更新和删除操作（CRUD）。下面是一些高层次的代码，开始实现游戏和这个逻辑：

```
class Game {
    private _gameState: any;

    public CurrentPlayerIndex: number;
    public Players: Player[];

    constructor(initialGameState: any) { this._gameState = initialGameState;}

}

class Player {
    private _playerState: any;

    public PlayerName: string;
    public PlayerScore: number;

    constructor(initialPlayerState: any) { this._playerState = initialPlayerState; }
}

class GameStateDBHelper {

    public CreateNewGame(): Game {
        // Initialize a new Game object and save it to the back end database.
        // Return the empty game.
        return new Game(null);
    }

    public LoadGame(query: string): Game {
        // Use supplied query to load some game state from the database.
        // Convert that to a GameState object
        // return it
        return new Game(null);
    }

    public SaveGame(gameToSave: Game): boolean {
        // Marshall game state and save it to the back end database.
        return true; // indicates successful save
    }

    public DeleteGame(gameToDelete: Game): boolean {
        // Issue database command to delete game state.
        return true; // indicates successful deletion
    }

}

const gameHelper = new GameStateDBHelper();
const newGame = gameHelper.CreateNewGame();
const oldGame = gameHelper.LoadGame("a database query");

const didSaveGame = gameHelper.SaveGame(oldGame);
const didDeleteGame = gameHelper.DeleteGame(newGame); 
```

代码定义了三个类：

+   `Game`：这是游戏对象本身，跟踪整个游戏状态，包括玩家列表和当前活动玩家。

+   `Player`：代表游戏中的玩家。玩家也有一些状态信息，尽管与 `Game` 不同。

+   `GameStateDBHelper`：一个实用类，提供输入/输出操作，并支持游戏对象的所有四个 CRUD 操作。

`GameStateDBHelper` 定义了四个公共方法，分别用于 CRUD 操作的每个操作。这些方法都接受常识性的输入参数，并返回常识性的结果。考虑 `LoadGame`：

```
 public LoadGame(query: string): Game {
        // Use supplied query to load some game state from the database.
        // Convert that to a GameState object
        // return it
        return new Game(null);
    } 
```

`LoadGame` 接收一个查询（类似于 "select * from Games..."）。它解析结果并返回一个新的 `Game` 对象。显然，在示例中进行了很多手势，但希望概念是清楚的。

DB 辅助对象使得根据需要执行 CRUD 操作变得容易：

```
const gameHelper = new GameStateDBHelper();
const newGame = gameHelper.CreateNewGame();
const oldGame = gameHelper.LoadGame("a database query");

const didSaveGame = gameHelper.SaveGame(oldGame);
const didDeleteGame = gameHelper.DeleteGame(newGame); 
```

尽管这种方法清晰且类型强大，但仍然存在问题。我们已经知道我们将要另外创建一个基于数据库的实体 - `Player`。如果我们简单地遵循当前的方法，我们最终会创建一个新的辅助函数，`PlayerStateDBHelper`。它必须提供相同的 CRUD 函数，并且每个函数的形式几乎与 GameState 的形式相同。在这种情况下，“形式”意味着：

+   查找数据库连接信息。

+   访问数据库

+   执行一些只在细节上略有不同的常见命令，从一个对象到另一个对象。

+   管理错误

+   返回成功/失败消息

我们可以使用 TypeScript 的泛型功能来减轻大部分问题。下面是它的实现方式：

```
interface DBBackedEntity {
    TableName: string;
}

class GameState implements DBBackedEntity {
    private myDBTableName: string;
    public get TableName(): string { return this.myDBTableName; }

    public CurrentPlayerIndex: number;
    public AllPlayers: GamePlayer[];

    constructor(someGameState: any) { 
        this.myDBTableName = "Games";
    }

}

class GamePlayer implements DBBackedEntity {
    private myDBTableName: string;
    public get TableName(): string { return this.myDBTableName; }

    public PlayerName: string;
    public Score: number;

    constructor(somePlayerState: any) { 
        this.myDBTableName = "Players";
    }
}

class DBHelper<T extends DBBackedEntity> {
    public CreateRecord() : T { return null; }
    public ReadRecord(query: any): T { return null; }
    public DeleteRecord(basedOn: T): boolean { return true; }
    public UpdateRecord(basedOn: T) : boolean { return true; }
}

const gameStateHelper = new DBHelper<GameState>();
const gamePlayerHelper = new DBHelper<GamePlayer>();

const newPlayer = gamePlayerHelper.CreateRecord();
console.log(`New player score: ${newPlayer.Score}.`)

const existingGameState = gameStateHelper.ReadRecord("some query goes here");
const newGameState = gameStateHelper.CreateRecord();

const deleteResult = gameStateHelper.DeleteRecord(newGameState);
const updateResult = gameStateHelper.UpdateRecord(existingGameState); 
```

泛型引入了一些新的语法，并以新的方式利用了现有概念（如接口）。

代码首先定义了一个新的接口，`DBBackedEntity`。该接口需要一个文本字段，“TableName”。这显然通过名称将其映射到数据库表。

然后为游戏及其玩家分别创建两个模型。它们都实现了 DBBackedEntity 接口，并通过对象的构造函数分配了数据库表名。

`DBHelper` 类引入了泛型语法：

```
class DBHelper<T extends DBBackedEntity> {
    public CreateRecord() : T { return null; }
    public ReadRecord(query: any): T { return null; }
    public DeleteRecord(basedOn: T): boolean { return true; }
    public UpdateRecord(basedOn: T) : boolean { return true; }
} 
```

这个语法，`<T extends DBBackedEntity>`有效地表示，“DB 助手类适用于任何实现 DBBackedEntity 接口的类型（类）”。当客户端代码实例化 DBHelper 的实例时，它将为该参数`T`指定一个值。这两行展示了如何为`T`传递一个值：

```
const gameStateHelper = new DBHelper<GameState>();
const gamePlayerHelper = new DBHelper<GamePlayer>(); 
```

在使用泛型时，我们通过尖括号提供类型参数：`DBHelper<GameState>`和`DBHelper<GamePlayer>`。TypeScript 分别用`GameState`和`GamePlayer`替换`DBHelper`类中的`T`参数。

# 进一步阅读

+   我写了一篇详尽的博客文章，描述了如何使用泛型来实现二分查找。你可以在这里阅读：[`blog.hellojs.org/implement-binary-search-in-typescript-using-generics-with-useful-refactorings-a4bcda932d7`](https://blog.hellojs.org/implement-binary-search-in-typescript-using-generics-with-useful-refactorings-a4bcda932d7)。

+   对 React 开发者特别感兴趣的是，这篇文章描述了如何在泛型中使用默认值：[`blog.mariusschulz.com/2017/06/02/typescript-2-3-generic-parameter-defaults`](https://blog.mariusschulz.com/2017/06/02/typescript-2-3-generic-parameter-defaults)。请注意，它是在 React 应用程序的背景下编写的，但该功能并不与 React 绑定。

# 总结

这一章关于泛型将*另一本 TypeScript 书*的主体部分结束了。下一章建议一些你可能感兴趣的额外阅读和视频。
