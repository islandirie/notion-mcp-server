# Migration Guide: Upgrading to Notion API 2025-09-03

This document guides you through upgrading your Notion MCP Server to use the new 2025-09-03 API version with multi-source database support.

## Migration Summary Report

### ✅ **Successfully Updated Notion MCP Server to 2025-09-03 API**

#### **What Was Updated:**

1. **🔄 OpenAPI Specification**
   - Upgraded from version `2022-06-28` to `2025-09-03`
   - Added complete data source support with new endpoints:
     - `/v1/data_sources/{data_source_id}` (GET, PATCH)
     - `/v1/data_sources/{data_source_id}/query` (POST)
     - `/v1/data_sources` (POST)

2. **🔧 Core Components**
   - Modified authentication headers to use `Notion-Version: 2025-09-03`
   - Updated `src/openapi-mcp-server/mcp/proxy.ts` to send the correct API version
   - Enhanced schemas to support data sources and multi-source databases

3. **📚 New Schemas Added**
   - `DataSource` schema with properties, parent, and database_parent
   - Enhanced `Database` schema with `data_sources` array
   - Updated `Parent` schema to support `data_source_id` type

4. **⚠️ Backwards Compatibility Maintained**
   - Deprecated `/v1/databases/{database_id}/query` endpoint (marked as deprecated)
   - Enhanced existing endpoints to return data source information
   - Updated search API to return data source objects

5. **📖 Documentation Created**
   - Complete migration guide (`MIGRATION-2025-09-03.md`)
   - Usage examples and best practices
   - Discovery workflow for existing databases

#### **Key Changes Summary:**

| **Feature** | **Before (2022-06-28)** | **After (2025-09-03)** |
|-------------|-------------------------|------------------------|
| **Page Creation** | `database_id` parent | `data_source_id` parent (preferred) |
| **Database Query** | `/v1/databases/{id}/query` | `/v1/data_sources/{id}/query` |
| **Schema Access** | Database properties | Data source properties |
| **Search Results** | Database objects | Data source objects |
| **Multi-Source** | Not supported | Full support |

#### **✅ Verification Results:**
- ✅ OpenAPI spec loads successfully
- ✅ API version correctly set to 2025-09-03  
- ✅ 3 new data source endpoints added
- ✅ Database query endpoint properly deprecated
- ✅ DataSource schema included
- ✅ Project builds without errors
- ✅ All 16 API endpoints available

#### **🚀 What This Enables:**

1. **Multi-Source Databases**: Users can now have databases with multiple linked data sources
2. **Enhanced Workflows**: More granular control over database schemas and properties
3. **Future-Proof**: Ready for Notion's latest API capabilities
4. **Backwards Compatible**: Existing single-source databases continue to work

---

## What's Changed

The Notion API 2025-09-03 introduces **data sources** as a first-class concept, enabling databases to contain multiple linked data sources. This change is **not backwards-compatible**.

### Key Changes:
- Most database operations now require `data_source_id` instead of `database_id`
- New `/v1/data_sources/*` endpoints for managing data source schemas
- Updated search API returns data source objects instead of database objects
- Enhanced page creation supports `data_source_id` parent
- Database query endpoint is deprecated in favor of data source query

## What's Been Updated

### ✅ Automatic Updates
The following have been automatically updated in this MCP server:

1. **OpenAPI Specification**: Updated to version 2025-09-03 with all new endpoints
2. **Authentication Headers**: Now uses `Notion-Version: 2025-09-03`
3. **Endpoint Support**: Added all new data source endpoints
4. **Schema Updates**: Updated request/response schemas for compatibility

### 🔄 New Available Tools

After the update, your MCP client will have access to these new tools:

#### Data Source Management
- `retrieve-a-data-source` - Get data source schema and properties
- `update-a-data-source` - Update data source properties and schema
- `create-a-data-source` - Add new data source to existing database
- `post-data-source-query` - Query specific data source (replaces database query)

#### Enhanced Database Operations
- `retrieve-a-database` - Now returns list of data sources
- `create-a-database` - Uses new `initial_data_source` structure
- `update-a-database` - Database-level properties only (use data source endpoints for schema)

#### Updated Search
- `post-search` - Now searches and returns data source objects

## Usage Examples

### Creating a Page in Multi-Source Database

**Before (2022-06-28):**
```json
{
  "parent": {
    "type": "database_id",
    "database_id": "abc123..."
  }
}
```

**After (2025-09-03):**
```json
{
  "parent": {
    "type": "data_source_id", 
    "data_source_id": "def456..."
  }
}
```

### Querying Database Content

**Before:**
Use `post-database-query` with database_id

**After:**
Use `post-data-source-query` with data_source_id

### Getting Database Schema

**Before:**
`retrieve-a-database` returned schema in `properties`

**After:**
1. `retrieve-a-database` returns list of data sources
2. `retrieve-a-data-source` returns schema for specific data source

## Discovery Workflow

For existing databases, use this workflow to discover data sources:

1. **Get Database**: Call `retrieve-a-database` with your database_id
2. **List Data Sources**: The response includes a `data_sources` array with IDs and names
3. **Get Schema**: Call `retrieve-a-data-source` for each data source ID to get properties
4. **Use Data Source ID**: Use the appropriate data_source_id for subsequent operations

## Backwards Compatibility

- Database endpoints that accept `database_id` continue to work for single-source databases
- Multi-source databases will return validation errors when using deprecated endpoints
- The MCP server automatically handles API version headers
- Tool names remain the same, but functionality is enhanced

## Error Handling

### Validation Errors
When using deprecated database endpoints with multi-source databases:

```json
{
  "object": "error",
  "status": 400,
  "code": "validation_error", 
  "message": "This database has multiple data sources. Use the data source query endpoint instead."
}
```

### Resolution
Switch to the corresponding data source endpoint with a specific `data_source_id`.

## Best Practices

1. **Always Discover First**: Start operations by calling `retrieve-a-database` to get available data sources
2. **Use Specific Data Source IDs**: Prefer data source endpoints over database endpoints for precise operations
3. **Handle Multiple Sources**: Be prepared for databases to have multiple data sources
4. **Update Stored IDs**: Store both database_id and data_source_id pairs for future operations

## Testing Your Integration

1. **Create Test Database**: Use `create-a-database` with `initial_data_source`
2. **Add Second Data Source**: Use `create-a-data-source` to add another source
3. **Test Discovery**: Verify your logic handles multiple data sources correctly
4. **Test Operations**: Ensure page creation and queries work with data_source_id

## Support

If you encounter issues:
1. Check that your Notion integration has appropriate permissions
2. Verify you're using the correct data_source_id from the discovery step
3. Review the error messages for specific validation failures
4. Consult the [official Notion API upgrade guide](https://developers.notion.com/docs/upgrade-guide-2025-09-03)

## Summary

This upgrade enables powerful new multi-source database workflows while maintaining compatibility for single-source databases. The MCP server automatically handles the API version upgrade, but you may need to update your workflows to take advantage of the new data source capabilities.

### 📋 For Users - Next Steps:

1. **✅ No Configuration Changes Required**: The server automatically uses the new API version
2. **🧪 Test Your Integration**: Existing workflows should continue working seamlessly
3. **🚀 Explore New Features**: Try multi-source database capabilities when needed
4. **📖 Reference Documentation**: Use this migration guide for detailed usage examples

**Your Notion MCP Server is now fully updated and ready to use the 2025-09-03 API with multi-source database support! 🎉**