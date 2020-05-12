---
description: 版本：11
---

# 1.1. 安裝

你需要先進行安裝，才能開始使用 PostgreSQL。當然，PostgreSQL 也可能已經被安裝在你的系統之中，因為你的作業系統預設套件包含了 PostgreSQL，或其他系統管理者已先行安裝。如果是這樣的話，那麼你應該先瞭解作業系統的資訊，或向你的系統管理員取得存取方式的資訊。

如果你並不確定 PostgreSQL 是否已經可以使用，那麼你也可以自行安裝試試。這樣做並不是很困難，而且是很好的操作練習。PostgreSQL 可以以一般使用者進行安裝，它並不需要系統管理者（root）的權限才能安裝。

如果你打算自行安裝 PostgreSQL，你可以參考[第 16 章](../../server-administration/installation-from-source-code/)的指令進行，完成之後再回到這裡，以瞭解下面有關設定環境變數的內容。

如果你的系統管理者並非以預設的方式安裝，你可能還有一些額外的工作要做。例如，如果資料庫主機其實是遠端的伺服器，你會需要設定 PGHOST 的環境變數，將其指向資料庫主機的網路名稱。而 PGPORT 變數也是必須要設定的。最基本的情境是，如果你嘗試啓動一個應用程式，而它回報它無法取得資料庫連線時，你就必須洽詢你的系統管理者。而如果系統管理者就是你自己，那麼你應該依文件再確認你的環境設定是正確的。如果你仍然並不清楚前面所描述的事項，請詳細閱讀下一節的內容。
