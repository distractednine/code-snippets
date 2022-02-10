public void SaveDataWithTransaction(ConnectionData connectionData, Guid productID, UserFavourites[] usersFavourites, Action progressUpdateAction)
        {
            using var companyDBContext = new CompanyDbContext(connectionData.ConnectionString);
            var strategy = companyDBContext.Database.CreateExecutionStrategy();

            strategy.Execute(() =>
            {
                using var transaction = companyDBContext.Database.BeginTransaction();
                try
                {
                    SaveDataToCompanyDbContext(companyProxyDBContext, connectionData.CompanyID, productID, usersFavourites);
                    companyDBContext.SaveChanges();

                    progressUpdateAction();
                    transaction.Commit();
                }
                catch (Exception ex)
                {
                    transaction.Rollback();
                    logger.LogError(ex, "Error during saving data occurred");
                }
            });
        }


// see: https://dzfweb.gitbooks.io/microsoft-microservices-book/content/implement-resilient-applications/implement-resilient-entity-framework-core-sql-connections.html

// see https://docs.microsoft.com/en-us/ef/core/miscellaneous/connection-resiliency